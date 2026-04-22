# Patrones Arquitectónicos — NN Auth System

<!--
  Archivo: patrones-arquitectonicos.md
  Descripcion: Documentacion tecnica ilustrada de los patrones arquitectonicos
               aplicados en el proyecto NN Auth System.
  Para que? Servir como referencia de estudio y consulta para entender por que
            el sistema esta estructurado como lo esta.
  Impacto: Comprender los patrones facilita mantener, extender y defender
           decisiones tecnicas del proyecto ante evaluaciones o presentaciones.
-->

> **Proyecto:** NN Auth System  
> **Stack:** Express.js + React + MongoDB + Docker

---

## Resumen ejecutivo

El sistema aplica **10 patrones arquitectónicos y de diseño** de uso profesional. No son solo teoría: cada patrón resuelve un problema concreto y está presente en el código del proyecto.

| #   | Patrón                     | Dónde vive                                | Qué resuelve                                         |
| --- | -------------------------- | ----------------------------------------- | ---------------------------------------------------- |
| 1   | Arquitectura en Capas      | `be/src/`                                 | Separación de responsabilidades en el backend        |
| 2   | DTO — Data Transfer Object | `be/src/types/` + `fe/src/types/`         | Nunca exponer datos internos de BD en respuestas     |
| 3   | Middleware Chain            | `be/src/middleware/`                      | Desacoplar validación, auth y logging del handler    |
| 4   | JWT Stateless              | `be/src/utils/security.ts`                | Autenticación sin estado en el servidor              |
| 5   | Context / Provider         | `AuthContext.tsx`                         | Estado de auth global en toda la app React           |
| 6   | Custom Hook                | `useAuth.ts`                              | Encapsular y reutilizar lógica de autenticación      |
| 7   | Interceptor                | `fe/src/api/auth.ts`                      | Adjuntar token JWT en cada petición automáticamente  |
| 8   | SPA + Route Guard          | `ProtectedRoute.tsx`                      | Proteger rutas sin renderizar páginas no autorizadas |
| 9   | Monorepo                   | `be/` + `fe/`                             | Código fuente unificado en un solo repositorio       |
| 10  | REST API                   | `be/src/routes/auth.routes.ts`            | Interfaz estándar entre frontend y backend           |

---

## Vista general del sistema

![](../assets/pa-01-overview.svg)

El sistema sigue una **arquitectura Cliente–Servidor** de tres capas lógicas:

1. **Frontend (React)** — Interfaz de usuario. Nunca guarda estado en el servidor.
2. **Backend (Express.js)** — Lógica de negocio. Expone una API REST bajo `/api/v1/`.
3. **Base de datos (MongoDB)** — Persistencia. Solo accedida desde el backend vía Mongoose.

La comunicación entre frontend y backend es exclusivamente **HTTP + JSON**. Los tokens JWT viajan en el header `Authorization: Bearer <token>`. Nunca hay sesiones en el servidor.

---

## Patrón 1 — Arquitectura en Capas

![](../assets/pa-02-backend-layers.svg)

### ¿Qué es?

Organizar el código en capas horizontales donde **cada capa solo puede comunicarse con la capa directamente inferior**.

### ¿Cómo se aplica aquí?

```
HTTP Request
      ↓
┌─────────────────────────────────────────┐
│  routes/           → Capa HTTP          │  Define endpoints y middleware chain
├─────────────────────────────────────────┤
│  controllers/      → Capa de Handlers   │  Maneja req/res HTTP
├─────────────────────────────────────────┤
│  services/         → Capa de Negocio    │  Reglas y decisiones
├─────────────────────────────────────────┤
│  models/           → Capa de Datos      │  Mongoose + Validación
├─────────────────────────────────────────┤
│  utils/            → Capa Transversal   │  security · email
└─────────────────────────────────────────┘
      ↓
MongoDB
```

### Ejemplo en código

```typescript
// routes/auth.routes.ts — solo define el endpoint y la cadena de middleware
router.post('/register', validate(registerRules), authController.register);

// controllers/auth.controller.ts — maneja req/res y delega al service
export async function register(req: Request, res: Response): Promise<void> {
  const user = await authService.registerUser(req.body);  // delega al service
  res.status(201).json(user);
}

// services/auth.service.ts — contiene la lógica de negocio
export async function registerUser(data: RegisterRequest): Promise<UserResponse> {
  const existing = await User.findOne({ email: data.email });
  if (existing) throw new AppError(400, 'El email ya está registrado');
  data.password = await hashPassword(data.password);  // llama a utils
  // ...
}
```

### Ventaja

Un cambio en la base de datos **no afecta** el router. Un cambio en el router **no afecta** la lógica de negocio. Cada capa es testeable de forma independiente.

---

## Patrón 2 — DTO (Data Transfer Object)

![](../assets/pa-04-dto-pattern.svg)

### ¿Qué es?

Un objeto diseñado exclusivamente para transportar datos entre capas, diferente del modelo de base de datos.

### ¿Por qué es crítico aquí?

El modelo Mongoose `User` tiene el campo `hashedPassword`. Si devolviéramos el documento directamente, **el hash de la contraseña quedaría expuesto en la respuesta HTTP**. El DTO actúa como filtro.

### Ejemplo en código

```typescript
// models/User.ts — Documento Mongoose (lo que hay en la BD)
interface IUser extends Document {
  email: string;
  firstName: string;
  lastName: string;
  hashedPassword: string;  // ← NUNCA debe salir en la respuesta
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// types/auth.ts — DTO (lo que se devuelve al cliente)
export interface UserResponse {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  // hashedPassword: ← OMITIDO intencionalmente
  isActive: boolean;
  createdAt: string;
}
```

```typescript
// controllers/users.controller.ts — construye el DTO manualmente
export async function getMe(req: Request, res: Response): Promise<void> {
  const user = req.user as IUser;  // inyectado por auth.middleware.ts
  const response: UserResponse = {
    id: user._id.toString(),
    email: user.email,
    firstName: user.firstName,
    lastName: user.lastName,
    isActive: user.isActive,
    createdAt: user.createdAt.toISOString(),
    // hashedPassword jamás se incluye
  };
  res.json(response);
}
```

### Ventaja

La API puede cambiar su contrato (el tipo `UserResponse`) **sin alterar el schema de Mongoose**, y viceversa.

---

## Patrón 3 — Middleware Chain (Cadena de Middleware)

### ¿Qué es?

En Express, las funciones **middleware** son funciones que interceptan el ciclo request/response. Se encadenan con `next()` — cada middleware decide si procesa la petición, la modifica y la pasa al siguiente, o la corta con una respuesta de error.

### Ejemplo en código

```typescript
// middleware/auth.middleware.ts — verifica el JWT y agrega user al req
export async function requireAuth(
  req: Request,
  res: Response,
  next: NextFunction,
): Promise<void> {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    res.status(401).json({ error: 'Token requerido' });
    return;
  }
  try {
    const payload = verifyAccessToken(token);
    req.user = await User.findById(payload.userId);
    next();  // pasa al siguiente middleware o al controller
  } catch {
    res.status(401).json({ error: 'Token inválido o expirado' });
  }
}
```

```typescript
// routes/users.routes.ts — la cadena de middleware es explícita y legible
router.get('/me', requireAuth, usersController.getMe);
//                 ↑           ↑
//           middleware 1   controller (last step)
```

### Ventaja

La cadena de middleware hace que cada responsabilidad sea visible en la definición de la ruta. `requireAuth` es reutilizable en cualquier endpoint protegido sin duplicar código.

```typescript
// auth.routes.ts — composición explícita de middleware
router.post(
  '/change-password',
  requireAuth,                    // 1. verifica JWT
  validate(changePasswordRules),  // 2. valida inputs
  authController.changePassword,  // 3. lógica de negocio
);
```

---

## Patrón 4 — JWT Stateless

![](../assets/pa-03-jwt-flow.svg)

### ¿Qué es?

El servidor **no guarda sesión**. En cambio, emite un token firmado criptográficamente que el cliente presenta en cada request. El servidor solo verifica la firma.

### Tokens del sistema

| Token           | Duración       | Propósito                                                |
| --------------- | -------------- | -------------------------------------------------------- |
| `access_token`  | **15 minutos** | Autenticar cada request a endpoints protegidos           |
| `refresh_token` | **7 días**     | Obtener un nuevo `access_token` sin volver a hacer login |

### Ejemplo en código

```typescript
// utils/security.ts — creación del token
export function createAccessToken(payload: TokenPayload): string {
  return jwt.sign(payload, env.JWT_SECRET, { expiresIn: '15m', algorithm: 'HS256' });
}

// utils/security.ts — verificación del token
export function verifyAccessToken(token: string): TokenPayload {
  try {
    return jwt.verify(token, env.JWT_SECRET) as TokenPayload;
  } catch {
    throw new AppError(401, 'Token inválido o expirado');
  }
}
```

### Ventaja

El backend puede escalar a múltiples instancias sin compartir estado de sesión. No hay tabla de sesiones. La información del usuario viaja **dentro** del token.

---

## Patrones 5, 6, 7 y 8 — Frontend React

![](../assets/pa-05-react-patterns.svg)

---

## Patrón 5 — Context / Provider

### ¿Qué es?

React usa el patrón **Provider** para compartir estado global sin necesidad de pasar props manualmente por cada nivel del árbol de componentes.

### Ejemplo en código

```typescript
// context/AuthContext.tsx
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<UserResponse | null>(null);
  const [accessToken, setAccessToken] = useState<string | null>(null);

  const login = async (credentials: LoginRequest) => {
    const response = await authApi.login(credentials);
    setAccessToken(response.access_token);
    // ...
  };

  return (
    <AuthContext.Provider value={{ user, accessToken, login, logout, ... }}>
      {children}
    </AuthContext.Provider>
  );
}
```

```tsx
// main.tsx — AuthProvider envuelve toda la app
<AuthProvider>
  <BrowserRouter>
    <App />
  </BrowserRouter>
</AuthProvider>
```

### Ventaja

`DashboardPage`, `Navbar`, `ChangePasswordPage` **todos** acceden al mismo estado de autenticación sin recibir props.

---

## Patrón 6 — Custom Hook

### ¿Qué es?

Una función de React que encapsula lógica reutilizable y puede usar otros hooks internamente.

### Ejemplo en código

```typescript
// hooks/useAuth.ts
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth() debe usarse dentro de <AuthProvider>");
  }
  return context;
}
```

```typescript
// pages/DashboardPage.tsx — consumo del hook
export function DashboardPage() {
  const { user, logout } = useAuth();  // una línea, acceso completo al contexto
  return <h1>Bienvenido, {user?.full_name}</h1>;
}
```

### Ventaja

En lugar de escribir `useContext(AuthContext)` con su validación en cada componente, se centraliza en `useAuth()`. Si el contexto cambia, **solo se modifica el hook**.

---

## Patrón 7 — Interceptor

### ¿Qué es?

Middleware a nivel de cliente HTTP que procesa **todas** las peticiones/respuestas antes de que lleguen al código de la aplicación.

### Ejemplo en código

```typescript
// api/axios.ts
const api = axios.create({ baseURL: import.meta.env.VITE_API_URL });

// Interceptor de request — adjunta el token automáticamente
api.interceptors.request.use((config) => {
  const token = sessionStorage.getItem("access_token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Interceptor de response — maneja errores de autenticación
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      sessionStorage.clear();
      window.location.href = "/login";
    }
    return Promise.reject(error);
  },
);
```

### Ventaja

Ningún componente ni función de API necesita preocuparse por añadir el header `Authorization`. Si el token cambia de lugar (por ejemplo, de `sessionStorage` a una cookie), se modifica **un solo lugar**.

---

## Patrón 8 — SPA + Route Guard

### ¿Qué es?

En una SPA (_Single Page Application_), el enrutamiento ocurre en el cliente (JavaScript), sin recargar la página. El **Route Guard** protege rutas que requieren autenticación.

### Ejemplo en código

```typescript
// components/ProtectedRoute.tsx
export function ProtectedRoute() {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) return <LoadingSpinner />;

  // Si no está autenticado, redirige sin mostrar la página
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
}
```

```tsx
// App.tsx — configuración de rutas
<Routes>
  {/* Rutas públicas */}
  <Route path="/login" element={<LoginPage />} />
  <Route path="/register" element={<RegisterPage />} />

  {/* Rutas protegidas — require autenticación */}
  <Route element={<ProtectedRoute />}>
    <Route path="/dashboard" element={<DashboardPage />} />
    <Route path="/change-password" element={<ChangePasswordPage />} />
  </Route>
</Routes>
```

### Ventaja

Un usuario que visita `/dashboard` sin autenticarse **nunca ve** el HTML de la página. Es redirigido inmediatamente. No hay necesidad de protección en cada componente individualmente.

---

## Patrón 9 — Monorepo

### ¿Qué es?

Múltiples proyectos (frontend, backend, infraestructura) conviven en **un solo repositorio git**.

### Estructura

```
proyecto-mern/             ← Un solo repositorio git
├── be/                    ← Backend (Node.js/Express.js)
│   ├── src/
│   └── package.json
├── fe/                    ← Frontend (React/TypeScript)
│   ├── src/
│   └── package.json
├── docker-compose.yml     ← Infraestructura compartida
└── .github/
    └── copilot-instructions.md
```

### Ventaja

- Un `git clone` obtiene todo el proyecto
- Los cambios que afectan a backend **y** frontend viajan en el mismo commit
- La infraestructura (`docker-compose.yml`) es parte del código versionado

---

## Patrón 10 — REST API

### ¿Qué es?

Interfaz de comunicación basada en recursos HTTP con verbos (`GET`, `POST`, `PUT`, `DELETE`) y códigos de estado estándar (`200`, `201`, `400`, `401`, `404`, `422`).

### Endpoints del sistema

| Verbo  | Ruta                           | Código OK | Descripción                            |
| ------ | ------------------------------ | --------- | -------------------------------------- |
| `POST` | `/api/v1/auth/register`        | `201`     | Registrar nuevo usuario                |
| `POST` | `/api/v1/auth/login`           | `200`     | Iniciar sesión, obtener tokens         |
| `POST` | `/api/v1/auth/refresh`         | `200`     | Renovar access token                   |
| `POST` | `/api/v1/auth/change-password` | `200`     | Cambiar contraseña (auth)              |
| `POST` | `/api/v1/auth/forgot-password` | `200`     | Solicitar email de recuperación        |
| `POST` | `/api/v1/auth/reset-password`  | `200`     | Restablecer contraseña con token       |
| `GET`  | `/api/v1/users/me`             | `200`     | Obtener perfil del usuario autenticado |

### Ejemplo de respuesta estándar

```json
// POST /api/v1/auth/login → 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}

// POST /api/v1/auth/register → 400 Bad Request (validación fallida)
{
  "errors": [
    { "field": "email", "message": "Debe ser un email válido" }
  ]
}
```

### Ventaja

Cualquier cliente (React, Android, iOS, Postman, curl) puede consumir la API porque habla HTTP estándar. La documentación está disponible en [`docs/referencia-tecnica/api-endpoints.md`](../referencia-tecnica/api-endpoints.md).

---

## Relación entre patrones

```
┌─────────────────────────────────────────────────────────────────┐
│ Monorepo (#9)                                                   │
│                                                                 │
│  ┌─── REST API (#10) ────────────────────────────────────────┐  │
│  │                                                           │  │
│  │  Frontend (SPA #8)          Backend (Capas #1)            │  │
│  │  ┌────────────────────┐     ┌──────────────────────────┐  │  │
│  │ Provider (#5)      │     │ routes/                  │  │  │
│  │  Hook (#6)         │←────│ services/   ← MW (#3)    │  │  │
│  │  RouteGuard (#8)   │────→│ models/     ← DTO (#2)   │  │  │
│  │  Interceptor (#7)  │     │ utils/      ← JWT (#4)   │  │  │
│  └────────────────────┘     └──────────────────────────┘  │  │
│                                        ↕                  │  │
│                               MongoDB (Mongoose)           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

Cada patrón resuelve un problema específico. Juntos, hacen que el sistema sea:

- **Seguro** — DTO + JWT + bcryptjs
- **Mantenible** — Capas + Middleware + CustomHook
- **Escalable** — Stateless + REST + Monorepo
- **Testeable** — mongodb-memory-server + supertest
