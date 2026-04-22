# 🎓 Instrucciones del Proyecto — NN Auth System (MERN)

<!--
  ¿Qué? Archivo de instrucciones para GitHub Copilot y colaboradores del proyecto.
  ¿Para qué? Define TODAS las reglas, convenciones, tecnologías y estándares que se
  deben seguir en cada archivo, commit, test y decisión técnica del proyecto.
  ¿Impacto? Garantiza consistencia, calidad y enfoque pedagógico en todo el código generado.
  Este archivo es la "ley" del proyecto — todo lo que se haga debe alinearse con estas reglas.
-->

---

## 1. Identidad del Proyecto

| Campo               | Valor                                                                                                                   |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Nombre**          | NN Auth System — MERN                                                                                                   |
| **Tipo**            | Proyecto educativo — SENA                                                                                               |
| **Propósito**       | Sistema de autenticación completo (registro, login, cambio y recuperación de contraseña) para una empresa genérica "NN" |
| **Enfoque**         | Aprendizaje guiado: cada línea de código y documentación debe enseñar                                                   |
| **Stack**           | MERN — MongoDB, Express.js, React, Node.js                                                                              |
| **Fecha de inicio** | Abril 2026                                                                                                              |

---

## 2. Stack Tecnológico

### 2.1 Backend (`be/`)

| Tecnología        | Versión | Propósito                                              |
| ----------------- | ------- | ------------------------------------------------------ |
| Node.js           | 20 LTS+ | Runtime de JavaScript — lenguaje principal del backend |
| TypeScript        | 5.0+    | Superset tipado de JavaScript (obligatorio en BE)      |
| Express.js        | 4.x     | Framework HTTP minimalista para Node.js                |
| Mongoose          | 8.x     | ODM para interactuar con MongoDB                       |
| jsonwebtoken      | 9.x     | Creación y verificación de tokens JWT                  |
| bcryptjs          | 2.x     | Hashing seguro de contraseñas (implementación pura JS) |
| express-validator | 7.x     | Validación y sanitización de inputs en Express         |
| nodemailer        | 6.x+    | Envío de emails (recuperación de contraseña)           |
| cors              | 2.x     | Middleware CORS para Express                           |
| helmet            | 8.x     | Cabeceras HTTP de seguridad                            |
| morgan            | 1.x     | Logger HTTP para desarrollo                            |
| dotenv            | 16.x    | Carga de variables de entorno desde `.env`             |
| jest              | 29.x    | Framework de testing                                   |
| ts-jest           | 29.x    | Soporte TypeScript para Jest                           |
| supertest         | 7.x     | Cliente HTTP para tests de integración                 |
| ESLint            | 9.x     | Linter para TypeScript                                 |
| Prettier          | 3.x     | Formateador de código                                  |
| tsx               | 4.x     | Ejecutor TypeScript para desarrollo (`tsx watch`)      |

### 2.2 Frontend (`fe/`)

| Tecnología                       | Versión | Propósito                                    |
| -------------------------------- | ------- | -------------------------------------------- |
| Node.js                          | 20 LTS+ | Runtime de JavaScript                        |
| React                            | 18+     | Biblioteca para interfaces de usuario        |
| Vite                             | 6+      | Bundler y dev server ultrarrápido            |
| TypeScript                       | 5.0+    | Superset tipado de JavaScript                |
| TailwindCSS                      | 4+      | Framework CSS utility-first                  |
| React Router                     | 7+      | Enrutamiento del lado del cliente            |
| Axios                            | latest  | Cliente HTTP para comunicación con la API    |
| Vitest                           | latest  | Framework de testing compatible con Vite     |
| Testing Library                  | latest  | Utilidades de testing para componentes React |
| ESLint                           | latest  | Linter para TypeScript/React                 |
| Prettier                         | latest  | Formateador de código                        |
| i18next                          | latest  | Motor de internacionalización (i18n)         |
| react-i18next                    | latest  | Integración de i18next con React (hooks/HOC) |
| i18next-browser-languagedetector | latest  | Detecta idioma del navegador automáticamente |

### 2.3 Base de Datos

| Tecnología     | Versión | Propósito                                            |
| -------------- | ------- | ---------------------------------------------------- |
| MongoDB        | 7+      | Base de datos documental principal (NoSQL)           |
| Docker Compose | latest  | Orquestación de contenedores (MongoDB en desarrollo) |

### 2.4 Autenticación

| Concepto      | Implementación                                                |
| ------------- | ------------------------------------------------------------- |
| Método        | JWT (JSON Web Tokens) — stateless                             |
| Access Token  | Duración: 15 minutos                                          |
| Refresh Token | Duración: 7 días                                              |
| Hashing       | bcrypt vía bcryptjs                                           |
| Flujos        | Registro, Login, Cambio de contraseña, Recuperación por email |

---

## 3. Reglas de Lenguaje — OBLIGATORIAS

### 3.1 Nomenclatura técnica → INGLÉS

Todo lo que sea código debe estar en **inglés**:

- Variables, funciones, clases, métodos
- Nombres de archivos y carpetas de código
- Endpoints y rutas de la API
- Nombres de tablas y columnas en la base de datos
- Nombres de componentes React
- Mensajes de commits
- Ramas de git

```python
# ✅ CORRECTO
def get_user_by_email(email: str) -> User:
    ...

# ❌ INCORRECTO
def obtener_usuario_por_email(correo: str) -> Usuario:
    ...
```

### 3.2 Comentarios y documentación → ESPAÑOL

Todo lo que sea documentación o comentarios debe estar en **español**:

- Comentarios en el código (`#`, `//`, `/* */`)
- Docstrings de funciones y clases
- Archivos de documentación (`.md`)
- README.md
- Descripciones en archivos de configuración
- Comentarios JSDoc en TypeScript

### 3.3 Regla del comentario pedagógico — ¿QUÉ? ¿PARA QUÉ? ¿IMPACTO?

**Cada comentario significativo debe responder tres preguntas:**

```typescript
/**
 * ¿Qué? Función que hashea la contraseña del usuario usando bcrypt.
 * ¿Para qué? Almacenar contraseñas de forma segura, nunca en texto plano.
 * ¿Impacto? Si se omite el hashing, las contraseñas quedan expuestas ante una filtración de la BD.
 */
async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}
```

```typescript
/**
 * ¿Qué? Hook personalizado que provee el estado de autenticación y sus acciones.
 * ¿Para qué? Centralizar la lógica de auth para que cualquier componente pueda consumirla.
 * ¿Impacto? Sin este hook, cada componente tendría que reimplementar la lógica de auth,
 * causando duplicación de código y posibles inconsistencias.
 */
export function useAuth(): AuthContextType {
  // ...
}
```

### 3.4 Cabecera de archivo obligatoria

Cada archivo nuevo debe incluir un **comentario de cabecera** al inicio:

```typescript
/**
 * Archivo: security.ts
 * Descripción: Utilidades de seguridad — hashing de contraseñas y manejo de tokens JWT.
 * ¿Para qué? Proveer funciones reutilizables de seguridad que se usan en todo el sistema de auth.
 * ¿Impacto? Es la base de la seguridad del sistema. Un error aquí compromete toda la autenticación.
 */
```

```typescript
/**
 * Archivo: AuthContext.tsx
 * Descripción: Contexto de React que gestiona el estado de autenticación global.
 * ¿Para qué? Proveer a toda la aplicación acceso al usuario autenticado, tokens y acciones de auth.
 * ¿Impacto? Sin este contexto, no habría forma de saber si el usuario está logueado
 * ni de proteger rutas que requieren autenticación.
 */
```

---

## 4. Reglas de Entorno y Herramientas — OBLIGATORIAS

### 4.1 Node.js — SIEMPRE usar `pnpm` (backend y frontend)

En el stack MERN, **tanto el backend como el frontend usan Node.js**. En ambos se usa exclusivamente `pnpm`:

```bash
# ✅ CORRECTO — be/ y fe/
pnpm install
pnpm add express@4.21.2
pnpm add -D jest@29.7.0
pnpm dev
pnpm test
pnpm build

# ❌ INCORRECTO — NUNCA usar npm
npm install        # ← PROHIBIDO
npm run dev        # ← PROHIBIDO
npx some-tool      # ← Usar pnpm dlx en su lugar

# ❌ INCORRECTO — NUNCA usar yarn
yarn install       # ← PROHIBIDO
```

Si algún tutorial o documentación sugiere `npm`, **reemplazar** por el equivalente `pnpm`.

### 4.2 Versiones de dependencias — PINEAR SIEMPRE (REGLA DE ORO)

> **"Una versión no pineada es una vulnerabilidad en espera de manifestarse."**

**PROHIBIDO** usar rangos de versión en `package.json` (backend ni frontend):

| Operador          | Significado               | Por qué está prohibido                        |
| ----------------- | ------------------------- | --------------------------------------------- |
| `^1.2.3`          | cualquier `1.x.x` ≥ 1.2.3 | Permite instalar 1.99.0 con CVEs desconocidos |
| `~1.2.3`          | cualquier `1.2.x` ≥ 1.2.3 | Permite parches que rompen compatibilidad     |
| `>=1.2.3`         | cualquier versión ≥ 1.2.3 | Sin límite superior — completamente inseguro  |
| `>1.2.3`          | cualquier versión > 1.2.3 | Ídem — sin límite superior                    |
| `*`               | cualquier versión         | La peor opción — literalmente ruleta rusa     |
| ` ` (sin versión) | última disponible         | Sin trazabilidad ni reproducibilidad          |

**OBLIGATORIO** usar versiones exactas en todo momento:

```json
// ✅ CORRECTO — package.json (be/ o fe/)
{
  "dependencies": {
    "express": "4.21.2",
    "mongoose": "8.13.2",
    "jsonwebtoken": "9.0.2",
    "bcryptjs": "2.4.3"
  }
}

// ❌ INCORRECTO — package.json
{
  "dependencies": {
    "express": "^4.21.2",
    "mongoose": "~8.13.0",
    "jsonwebtoken": "*"
  }
}
```

**Flujo obligatorio al agregar o actualizar una dependencia:**

```bash
# Node.js — SIEMPRE con @X.Y.Z (backend y frontend)
pnpm add paquete@X.Y.Z                 # pnpm respeta la versión exacta
pnpm audit                             # Verificar CVEs
# pnpm.overrides en package.json para dependencias transitivas con CVEs
```

**Dependencias transitivas vulnerables → usar `pnpm.overrides`:**

```json
// package.json
{
  "pnpm": {
    "overrides": {
      "vulnerable-pkg@<safe_version": "safe_version"
    }
  }
}
```

**Herramientas de auditoría de CVEs:**

```bash
# Backend y Frontend
pnpm audit                             # Audita deps del node_modules
pnpm audit --audit-level=high          # Solo reporta severidad high/critical
```

### 4.3 Variables de entorno

- **NUNCA** hardcodear credenciales, URIs de base de datos, secrets, o configuración sensible
- Usar archivos `.env` (no versionados en git)
- Proveer **siempre** un `.env.example` con las variables necesarias y valores de ejemplo
- Validar las variables de entorno al iniciar la aplicación (con `envalid` o similar en BE)

```bash
# be/.env.example
NODE_ENV=development
PORT=5000
MONGODB_URI=mongodb://localhost:27017/nn_auth_db
JWT_SECRET=your-super-secret-key-change-in-production-min-32-chars
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USER=
MAIL_PASS=
MAIL_FROM=noreply@nn-company.com
FRONTEND_URL=http://localhost:5173
```

---

## 5. Estructura del Proyecto

```
proyecto-mern/                     # Raíz del monorepo
├── .github/
│   └── copilot-instructions.md    # ← ESTE ARCHIVO — reglas del proyecto
├── .gitignore                     # Archivos ignorados por git
├── docker-compose.yml             # Servicios: MongoDB 7 + Mailpit
├── README.md                      # Documentación principal del proyecto
│
├── docs/                          # 📚 Documentación del proyecto
│   ├── referencia-tecnica/
│   │   ├── architecture.md        # Arquitectura general y diagramas
│   │   ├── api-endpoints.md       # Documentación de todos los endpoints
│   │   ├── database-schema.md     # Esquema de colecciones Mongoose
│   │   └── design-system.md       # Sistema de temas y tokens de color
│   ├── conceptos/                 # Conceptos educativos (OWASP, ARIA, etc.)
│   ├── requisitos/                # Historias de usuario y requisitos funcionales
│   └── setup/                     # Guías de instalación paso a paso
│
├── assets/                        # 🖼️ Recursos estáticos (imágenes, diagramas)
│
├── be/                            # 🟢 Backend — Express.js + TypeScript
│   ├── .env                       # Variables de entorno (NO versionado)
│   ├── .env.example               # Plantilla de variables de entorno
│   ├── package.json               # Dependencias y scripts
│   ├── pnpm-lock.yaml             # Lockfile de pnpm
│   ├── tsconfig.json              # Configuración de TypeScript
│   ├── eslint.config.js           # Configuración de ESLint
│   ├── jest.config.ts             # Configuración de Jest
│   └── src/
│       ├── index.ts               # Punto de entrada — inicia el servidor HTTP
│       ├── app.ts                 # Express app — registra middleware y rutas
│       ├── config/
│       │   └── env.ts             # Variables de entorno validadas (envalid)
│       ├── db/
│       │   └── connection.ts      # Conexión a MongoDB via Mongoose
│       ├── models/                # Modelos Mongoose (esquemas de colecciones)
│       │   ├── User.ts
│       │   └── PasswordResetToken.ts
│       ├── routes/                # Routers de Express (agrupan endpoints)
│       │   ├── auth.routes.ts     # Registro, login, refresh, password flows
│       │   └── users.routes.ts    # Perfil del usuario
│       ├── controllers/           # Controladores — manejan req/res HTTP
│       │   ├── auth.controller.ts
│       │   └── users.controller.ts
│       ├── services/              # Lógica de negocio (sin dependencia de HTTP)
│       │   └── auth.service.ts
│       ├── middleware/            # Middleware de Express
│       │   ├── auth.middleware.ts     # Verifica JWT y agrega user al req
│       │   └── validate.middleware.ts # Ejecuta validaciones de express-validator
│       ├── utils/                 # Utilidades transversales
│       │   ├── security.ts        # Hashing (bcryptjs) y JWT (jsonwebtoken)
│       │   └── email.ts           # Envío de emails (nodemailer)
│       └── __tests__/             # Tests
│           ├── setup.ts           # Configuración global de Jest (test DB)
│           └── auth.test.ts       # Tests de integración con supertest
│
└── fe/                            # ⚛️ Frontend — React + Vite + TypeScript
    ├── .env                       # Variables de entorno (NO versionado)
    ├── .env.example               # Plantilla de variables de entorno
    ├── index.html                 # HTML base de Vite
    ├── package.json               # Dependencias y scripts
    ├── pnpm-lock.yaml             # Lockfile de pnpm
    ├── vite.config.ts             # Configuración de Vite
    ├── tsconfig.json              # Configuración de TypeScript
    ├── eslint.config.js           # Configuración de ESLint
    └── src/
        ├── main.tsx               # Punto de entrada — renderiza App en el DOM
        ├── App.tsx                # Componente raíz — define rutas
        ├── index.css              # Estilos globales + imports de Tailwind
        ├── i18n.ts                # Configuración de i18next
        ├── locales/               # Catálogos de traducción
        │   ├── es/translation.json
        │   └── en/translation.json
        ├── api/                   # Clientes HTTP
        │   └── auth.ts            # Funciones para cada endpoint de auth
        ├── components/            # Componentes reutilizables
        │   ├── ui/                # Componentes UI genéricos (Button, Input, Alert)
        │   └── layout/            # Layout, Navbar, Footer
        ├── pages/                 # Páginas/vistas (una por ruta)
        │   ├── LandingPage.tsx
        │   ├── LoginPage.tsx
        │   ├── RegisterPage.tsx
        │   ├── DashboardPage.tsx
        │   ├── ChangePasswordPage.tsx
        │   ├── ForgotPasswordPage.tsx
        │   ├── ResetPasswordPage.tsx
        │   ├── ContactPage.tsx
        │   ├── TerminosDeUsoPage.tsx
        │   ├── PoliticaPrivacidadPage.tsx
        │   └── PoliticaCookiesPage.tsx
        ├── hooks/                 # Custom hooks
        │   └── useAuth.ts
        ├── context/               # React Context providers
        │   └── AuthContext.tsx
        ├── types/                 # Tipos e interfaces TypeScript
        │   └── auth.ts
        ├── utils/                 # Utilidades (formateo, validación, etc.)
        └── __tests__/             # Tests
            └── auth.test.tsx
```

---

## 6. Convenciones de Código

### 6.1 TypeScript (Backend — Express.js)

| Aspecto             | Convención                                                         |
| ------------------- | ------------------------------------------------------------------ |
| Estilo              | ESLint + Prettier                                                  |
| Naming variables    | `camelCase`                                                        |
| Naming clases       | `PascalCase` (modelos Mongoose, clases de error)                   |
| Naming constantes   | `UPPER_SNAKE_CASE`                                                 |
| Naming archivos     | `camelCase` para utilidades/services, `PascalCase` para modelos    |
| Tipos               | **Obligatorios** — nunca usar `any` sin justificación              |
| Interfaces vs Types | `interface` para shapes de objetos, `type` para uniones/utilidades |
| Imports             | Ordenados: Node built-ins → third-party → locales                  |
| Línea máxima        | 100 caracteres                                                     |

```typescript
// ✅ Ejemplo de función bien documentada y tipada (backend)
/**
 * ¿Qué? Verifica si una contraseña en texto plano coincide con su hash bcrypt.
 * ¿Para qué? Validar las credenciales del usuario durante el login.
 * ¿Impacto? Es el mecanismo central de autenticación — si falla, nadie puede loguearse.
 */
async function verifyPassword(
  plainPassword: string,
  hashedPassword: string,
): Promise<boolean> {
  return bcrypt.compare(plainPassword, hashedPassword);
}
```

### 6.2 TypeScript/React (Frontend)

| Aspecto             | Convención                                                            |
| ------------------- | --------------------------------------------------------------------- |
| Estilo              | ESLint + Prettier                                                     |
| Naming variables    | `camelCase`                                                           |
| Naming componentes  | `PascalCase`                                                          |
| Naming archivos     | `PascalCase` para componentes, `camelCase` para utilidades            |
| Naming tipos        | `PascalCase` con sufijo descriptivo (`UserResponse`, `LoginRequest`)  |
| Componentes         | **Funcionales** con hooks — nunca clases                              |
| Interfaces vs Types | Preferir `interface` para objetos, `type` para uniones/intersecciones |
| CSS                 | TailwindCSS utility classes — evitar CSS custom                       |
| Strict mode         | `"strict": true` en tsconfig.json                                     |

```typescript
// ✅ Ejemplo de componente bien documentado
/**
 * ¿Qué? Campo de entrada reutilizable con label, validación y mensajes de error.
 * ¿Para qué? Estandarizar todos los inputs del formulario de auth con un diseño consistente.
 * ¿Impacto? Sin este componente, cada formulario tendría su propia implementación de inputs,
 * resultando en inconsistencias visuales y duplicación de lógica de validación.
 */
interface InputFieldProps {
  label: string;
  name: string;
  type?: string;
  error?: string;
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
}

export function InputField({ label, name, type = "text", error, value, onChange }: InputFieldProps) {
  return (
    <div className="mb-4">
      <label htmlFor={name} className="block text-sm font-medium text-gray-700">
        {label}
      </label>
      <input
        id={name}
        name={name}
        type={type}
        value={value}
        onChange={onChange}
        className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
      />
      {error && <p className="mt-1 text-sm text-red-600">{error}</p>}
    </div>
  );
}
```

### 6.3 MongoDB / Mongoose

| Aspecto                | Convención                                                           |
| ---------------------- | -------------------------------------------------------------------- |
| Nombres de colecciones | `camelCase`, plural — Mongoose los pluraliza automáticamente         |
| Nombres de campos      | `camelCase` (`createdAt`, `hashedPassword`, `firstName`)             |
| Primary Key            | `_id` — ObjectId generado por MongoDB (default)                      |
| Referencias            | Campo `userId: ObjectId` con `ref: 'User'`                           |
| Timestamps             | `{ timestamps: true }` en el schema — genera `createdAt`/`updatedAt` |
| Validación             | Siempre en el schema Mongoose (`required`, `unique`, `enum`, etc.)   |
| **No hay migraciones** | Los cambios de esquema se gestionan con estrategias de evolución     |

```typescript
// ✅ Ejemplo de schema Mongoose bien definido
/**
 * ¿Qué? Schema Mongoose para el modelo User.
 * ¿Para qué? Definir la estructura, validaciones y comportamiento del documento de usuario en MongoDB.
 * ¿Impacto? Garantiza integridad de datos a nivel de aplicación (MongoDB es schemaless por defecto).
 */
const userSchema = new Schema<IUser>(
  {
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    firstName: { type: String, required: true, trim: true },
    lastName: { type: String, required: true, trim: true },
    hashedPassword: { type: String, required: true },
    isActive: { type: Boolean, default: true },
    isEmailVerified: { type: Boolean, default: false },
    locale: { type: String, enum: ["es", "en"], default: "es" },
  },
  { timestamps: true },
);
```

### 6.4 Internacionalización (i18n) — Frontend

> **Conceptos clave para los aprendices:**
>
> - **i18n** (internacionalización): preparar el código para admitir múltiples idiomas sin cambios de código
> - **l10n** (localización): adaptar el contenido al idioma y región específicos
> - **locale**: identificador de idioma/región (ej: `es`, `en`, `es-CO`, `en-US`)

| Aspecto                    | Convención                                                            |
| -------------------------- | --------------------------------------------------------------------- |
| Librería                   | `react-i18next` + `i18next` + `i18next-browser-languagedetector`      |
| Idiomas soportados         | `es` (español — por defecto) y `en` (inglés)                          |
| Archivos de traducción     | `src/locales/{locale}/translation.json`                               |
| Namespace                  | Un único namespace `translation` (simplicidad pedagógica)             |
| Claves de traducción       | **inglés**, `camelCase`, agrupadas por sección/página                 |
| Valores de traducción      | En el idioma correspondiente al archivo                               |
| Textos con variables       | Usar interpolación `{{variable}}` (ej: `"welcome": "Hola, {{name}}"`) |
| Almacenamiento preferencia | `localStorage` (clave `i18nextLng`) + columna `locale` en BD          |
| Detección automática       | `navigator.language` → fallback a `es`                                |

```typescript
// ✅ CORRECTO — Clave en inglés, sintaxis de hook

import { useTranslation } from "react-i18next";

function LoginPage() {
  const { t } = useTranslation();

  return <h1>{t("auth.login.title")}</h1>;
  // Renderiza: "Iniciar sesión" (en español) | "Sign in" (en inglés)
}

// ✅ CORRECTO — Interpolación de variables
function DashboardPage() {
  const { t } = useTranslation();
  const { user } = useAuth();

  return <h1>{t("dashboard.welcome", { name: user?.first_name })}</h1>;
  // Renderiza: "Bienvenido, Carlos" | "Welcome, Carlos"
}

// ❌ INCORRECTO — Texto hardcoded en componentes
function LoginPage() {
  return <h1>Iniciar sesión</h1>; // ← No se puede traducir
}
```

```json
// ✅ CORRECTO — Estructura de un archivo de traducción (es/translation.json)
{
  "auth": {
    "login": {
      "title": "Iniciar sesión",
      "submit": "Iniciar sesión"
    }
  },
  "dashboard": {
    "welcome": "Bienvenido, {{name}}"
  }
}
```

**Convenciones de estructura de claves:**

```
auth.login.title          → título de la página de login
auth.register.submit      → botón de submit del formulario de registro
auth.changePassword.*     → textos de la página cambio de contraseña
dashboard.*               → textos del dashboard
nav.*                     → textos de la navbar (brand, logout, etc.)
common.*                  → textos reutilizables (loading, cancel, etc.)
language.*                → textos del selector de idioma
```

---

## 7. Conventional Commits — OBLIGATORIO

### 7.1 Formato

```
type(scope): short description in english

What: Detailed description of what was done
For: Why this change is needed
Impact: What effect this has on the system
```

### 7.2 Tipos permitidos

| Tipo       | Uso                                                  |
| ---------- | ---------------------------------------------------- |
| `feat`     | Nueva funcionalidad                                  |
| `fix`      | Corrección de bug                                    |
| `docs`     | Solo documentación                                   |
| `style`    | Formato, espacios, puntos y comas (no afecta lógica) |
| `refactor` | Reestructuración sin cambiar funcionalidad           |
| `test`     | Agregar o corregir tests                             |
| `chore`    | Tareas de mantenimiento, configuración, dependencias |
| `ci`       | Cambios en CI/CD                                     |
| `perf`     | Mejoras de rendimiento                               |

### 7.3 Scopes sugeridos

- `auth` — Autenticación y autorización
- `user` — Modelo/funcionalidad de usuario
- `db` — Base de datos y migraciones
- `api` — Endpoints y routers
- `ui` — Componentes y estilos del frontend
- `config` — Configuración y entorno
- `test` — Tests
- `deps` — Dependencias

### 7.4 Ejemplos

```bash
# ✅ Ejemplo de commit completo
git commit -m "feat(auth): add user registration endpoint

What: Creates POST /api/v1/auth/register endpoint with email validation,
password hashing, and duplicate email check
For: Allow new users to create accounts in the NN Auth System
Impact: Enables the user onboarding flow; stores hashed passwords using
bcrypt in the users table"

# ✅ Ejemplo de fix
git commit -m "fix(auth): handle expired refresh token gracefully

What: Returns 401 with clear error message when refresh token is expired
For: Prevent confusing 500 errors when users try to refresh after 7 days
Impact: Improves UX by redirecting to login instead of showing error page"

# ✅ Ejemplo de chore
git commit -m "chore(deps): upgrade fastapi to 0.115.6

What: Updates FastAPI from 0.115.0 to 0.115.6 in requirements.txt
For: Include latest security patches and bug fixes
Impact: No breaking changes; all existing tests pass"
```

---

## 8. Calidad — NO es Opcional, es OBLIGACIÓN

### 8.1 Principio fundamental

> **Código que se genera, código que se prueba.**

Cada función, endpoint, componente o utilidad que se cree **debe** tener su test correspondiente.
No se considera "terminada" una feature hasta que sus tests pasen.

### 8.2 Testing — Backend

| Herramienta             | Propósito                                          |
| ----------------------- | -------------------------------------------------- |
| `jest`                  | Framework principal de testing                     |
| `ts-jest`               | Soporte TypeScript para Jest                       |
| `supertest`             | Cliente HTTP para tests de integración con Express |
| `mongodb-memory-server` | MongoDB en memoria para tests (sin Docker)         |

```bash
# Ejecutar todos los tests del backend
cd be && pnpm test

# Ejecutar con cobertura
pnpm test:coverage

# Ejecutar un test específico
pnpm test -- --testPathPattern=auth
```

**Cobertura mínima esperada**: 80% en módulos de lógica de negocio.

### 8.3 Testing — Frontend

| Herramienta              | Propósito                       |
| ------------------------ | ------------------------------- |
| `vitest`                 | Test runner compatible con Vite |
| `@testing-library/react` | Testing de componentes React    |
| `jsdom`                  | Simular el DOM en Node.js       |

```bash
# Ejecutar todos los tests del frontend
cd fe && pnpm test

# Ejecutar en modo watch
pnpm test:watch

# Ejecutar con cobertura
pnpm test:coverage
```

### 8.4 Linting y Formateo

```bash
# Backend — ESLint + Prettier
cd be && pnpm lint               # Verificar errores
cd be && pnpm format             # Formatear código

# Frontend — ESLint + Prettier
cd fe && pnpm lint               # Verificar errores
cd fe && pnpm format             # Formatear código
```

### 8.5 Checklist antes de commit

- [ ] ¿El código tiene tipos TypeScript en todo el código (BE y FE)?
- [ ] ¿Hay comentarios pedagógicos (¿Qué? ¿Para qué? ¿Impacto?)?
- [ ] ¿Los tests pasan? (`pnpm test` en be/ y fe/)
- [ ] ¿El linter no reporta errores? (`pnpm lint` en be/ y fe/)
- [ ] ¿El commit sigue Conventional Commits con What/For/Impact?
- [ ] ¿Las variables sensibles están en `.env` y no hardcodeadas?
- [ ] ¿El `.env.example` se actualizó si se agregaron nuevas variables?
- [ ] **Si se agregó/actualizó una dependencia:** ¿`pnpm audit` no reporta CVEs?
- [ ] **Si se agregó/actualizó una dependencia:** ¿La versión es exacta (sin `^` ni `~`) en `package.json`?

---

## 9. Seguridad — Mejores Prácticas

### 9.1 Contraseñas

- **SIEMPRE** hashear con bcrypt (vía `bcryptjs`) antes de almacenar
- **NUNCA** almacenar contraseñas en texto plano
- **NUNCA** loggear contraseñas ni incluirlas en responses
- Validar fortaleza mínima: ≥8 caracteres, al menos 1 mayúscula, 1 minúscula, 1 número

### 9.2 JWT (Tokens)

- Access Token: corta duración (15 min) — se envía en header `Authorization: Bearer <token>`
- Refresh Token: larga duración (7 días) — se usa solo para obtener nuevos access tokens
- Secret key: mínimo 32 caracteres, aleatoria, en variable de entorno
- Algoritmo: HS256
- **NUNCA** almacenar tokens en `localStorage` en producción (usar httpOnly cookies o memoria)

### 9.3 CORS

- Configurar orígenes permitidos explícitamente
- En desarrollo: permitir `http://localhost:5173`
- En producción: **NUNCA** usar `allow_origins=["*"]`

### 9.4 API

- Versionamiento: `/api/v1/...`
- Rate limiting en endpoints de auth (prevenir brute force) — `express-rate-limit`
- Validación de inputs con `express-validator` (nunca confiar en datos del cliente)
- Mensajes de error genéricos en auth (no revelar si el email existe)
- Cabeceras de seguridad con `helmet`

### 9.5 Base de datos

- Usar siempre Mongoose (nunca concatenar strings en queries — evitar NoSQL injection)
- Validar y sanitizar toda entrada antes de pasarla a Mongoose
- Credenciales en variables de entorno

---

## 10. Estructura de la API

### 10.1 Prefijo base

Todos los endpoints van bajo `/api/v1/`

### 10.2 Endpoints de autenticación (`/api/v1/auth/`)

| Método | Ruta               | Descripción                           | Auth requerida |
| ------ | ------------------ | ------------------------------------- | -------------- |
| POST   | `/register`        | Registrar nuevo usuario               | No             |
| POST   | `/login`           | Iniciar sesión, obtener tokens        | No             |
| POST   | `/refresh`         | Renovar access token con refresh      | No (\*)        |
| POST   | `/change-password` | Cambiar contraseña (usuario logueado) | Sí             |
| POST   | `/forgot-password` | Solicitar email de recuperación       | No             |
| POST   | `/reset-password`  | Restablecer contraseña con token      | No (\*)        |

(\*) Requiere un token válido (refresh o reset), pero no el access token estándar.

### 10.3 Endpoints de usuario (`/api/v1/users/`)

| Método | Ruta  | Descripción                       | Auth requerida |
| ------ | ----- | --------------------------------- | -------------- |
| GET    | `/me` | Obtener perfil del usuario actual | Sí             |

---

## 11. Esquema de Base de Datos

### 11.1 Colección `users`

| Campo             | Tipo     | Restricciones / Opciones                   |
| ----------------- | -------- | ------------------------------------------ |
| `_id`             | ObjectId | PK, auto-generado por MongoDB              |
| `email`           | String   | required, unique, indexed, lowercase, trim |
| `firstName`       | String   | required, trim                             |
| `lastName`        | String   | required, trim                             |
| `hashedPassword`  | String   | required                                   |
| `isActive`        | Boolean  | default: true                              |
| `isEmailVerified` | Boolean  | default: false                             |
| `locale`          | String   | enum: ['es', 'en'], default: 'es'          |
| `createdAt`       | Date     | auto — generado por `timestamps: true`     |
| `updatedAt`       | Date     | auto — generado por `timestamps: true`     |

### 11.2 Colección `passwordresettokens`

| Campo       | Tipo     | Restricciones / Opciones               |
| ----------- | -------- | -------------------------------------- |
| `_id`       | ObjectId | PK, auto-generado por MongoDB          |
| `userId`    | ObjectId | ref: 'User', required, indexed         |
| `token`     | String   | required, unique, indexed              |
| `expiresAt` | Date     | required                               |
| `used`      | Boolean  | default: false                         |
| `createdAt` | Date     | auto — generado por `timestamps: true` |

> **Nota pedagógica:** A diferencia de PostgreSQL, MongoDB no tiene migraciones.
> Los cambios de esquema se manejan con código de migración explícito o estrategias
> de evolución de documentos (backward-compatible fields, versioning, etc.).

---

## 12. Flujos de Autenticación

### 12.1 Registro

```
Cliente → POST /api/v1/auth/register { email, full_name, password }
  → Validar datos (Pydantic)
  → Verificar email no duplicado
  → Hashear password (bcrypt)
  → Crear usuario en BD
  → Retornar usuario creado (sin password)
```

### 12.2 Login

```
Cliente → POST /api/v1/auth/login { email, password }
  → Buscar usuario por email
  → Verificar password contra hash
  → Generar access_token (15 min) + refresh_token (7 días)
  → Retornar { access_token, refresh_token, token_type: "bearer" }
```

### 12.3 Cambio de contraseña (usuario autenticado)

```
Cliente → POST /api/v1/auth/change-password { current_password, new_password }
  → (Requiere Authorization: Bearer <access_token>)
  → Verificar current_password contra hash
  → Hashear new_password
  → Actualizar en BD
  → Retornar confirmación
```

### 12.4 Recuperación de contraseña (forgot + reset)

```
Paso 1: Solicitar recuperación
Cliente → POST /api/v1/auth/forgot-password { email }
  → Buscar usuario por email
  → Generar token de reset (UUID + expiración 1 hora)
  → Guardar token en tabla password_reset_tokens
  → Enviar email con enlace: {FRONTEND_URL}/reset-password?token={token}
  → Retornar mensaje genérico (no revelar si el email existe)

Paso 2: Restablecer contraseña
Cliente → POST /api/v1/auth/reset-password { token, new_password }
  → Buscar token en BD
  → Verificar que no haya expirado ni sido usado
  → Hashear new_password
  → Actualizar password del usuario
  → Marcar token como usado
  → Retornar confirmación
```

---

## 13. Configuración de Docker Compose

Servicios necesarios para desarrollo:

```yaml
# Solo para desarrollo local — MongoDB 7 + Mailpit
services:
  db:
    image: mongo:7-jammy
    container_name: nn_auth_mongo
    environment:
      MONGO_INITDB_DATABASE: nn_auth_db
    ports:
      - "27017:27017"
    volumes:
      - nn_auth_mongo_data:/data/db

  mailpit:
    image: axllent/mailpit
    container_name: nn_auth_mailpit
    ports:
      - "1025:1025" # SMTP
      - "8025:8025" # UI web

volumes:
  nn_auth_mongo_data:
```

---

## 14. Mejores Prácticas — Resumen

### 14.1 Generales

- ✅ **DRY** (Don't Repeat Yourself) — reutilizar código
- ✅ **KISS** (Keep It Simple, Stupid) — preferir soluciones simples
- ✅ **YAGNI** (You Aren't Gonna Need It) — no agregar lo que no se necesita aún
- ✅ **Separation of Concerns** — cada módulo tiene una responsabilidad clara
- ✅ **Fail fast** — validar inputs al inicio de cada operación
- ✅ **Defensive programming** — manejar errores explícitamente

### 14.2 Backend

- ✅ Separar routes (endpoints) de controllers (handlers) y services (lógica de negocio)
- ✅ Usar middleware de Express para validación, autenticación y logging
- ✅ Usar `express-validator` para toda validación de inputs de entrada
- ✅ Manejar errores con middleware centralizado de Express (`app.use(errorHandler)`)
- ✅ Definir interfaces TypeScript para todos los request/response bodies
- ✅ Usar `mongoose` schemas con validaciones estrictas (`strict: true` por defecto)

### 14.3 Frontend

- ✅ Componentes pequeños y reutilizables
- ✅ Estado global solo cuando es necesario (Context API para auth)
- ✅ Custom hooks para encapsular lógica reutilizable
- ✅ Rutas protegidas con componente `ProtectedRoute`
- ✅ Manejo de errores con feedback visual al usuario
- ✅ Loading states para operaciones asíncronas

### 14.4 Diseño y UX/UI — OBLIGATORIO

| Aspecto               | Regla                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Temas**             | Dark mode y Light mode con toggle — usar `prefers-color-scheme`    |
| **Tipografía**        | Fuentes **sans-serif** exclusivamente (`Inter`, `system-ui`)       |
| **Colores**           | Sólidos y planos — **SIN degradados** (`gradient`) en ningún lugar |
| **Estilo visual**     | Diseño moderno, limpio, minimalista con excelente UX/UI            |
| **Botones de acción** | Siempre alineados a la **derecha** (`justify-end`, `text-right`)   |
| **Spacing**           | Usar escala consistente de Tailwind (`p-4`, `gap-6`, `space-y-4`)  |
| **Bordes**            | Sutiles (`border`, `border-gray-200 dark:border-gray-700`)         |
| **Transiciones**      | Suaves en hover/focus (`transition-colors`, `duration-200`)        |
| **Responsividad**     | Mobile-first — los formularios de auth deben verse bien en móvil   |
| **Accesibilidad**     | Labels en inputs, `aria-*` básicos, contraste suficiente (WCAG AA) |

```typescript
// ✅ CORRECTO — Botón de acción a la derecha, sin degradados, sans-serif, accent token
<div className="flex justify-end gap-3">
  <button className="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300
    bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600
    rounded-lg hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors">
    Cancelar
  </button>
  <button className="px-4 py-2 text-sm font-medium text-white
    bg-accent-600 hover:bg-accent-700 dark:bg-accent-500 dark:hover:bg-accent-600
    rounded-lg transition-colors">
    Guardar
  </button>
</div>

// ❌ INCORRECTO — color concreto hardcodeado, degradados, botones centrados/izquierda
<div className="flex justify-center">
  <button className="bg-gradient-to-r from-blue-500 to-purple-500 font-serif">
    Guardar
  </button>
</div>
```

### 14.5 Sistema de Temas Diferenciales por Stack — OBLIGATORIO

Este proyecto pertenece a una **serie educativa** de múltiples repos que implementan
el mismo sistema con stacks backend diferentes. Cada proyecto tiene un color de acento
único asignado a su stack:

| Stack                        | Proyecto              | Color Tailwind | Rol en la serie     |
| ---------------------------- | --------------------- | -------------- | ------------------- |
| FastAPI (Python)             | `proyecto-be-fe`      | `emerald`      |                     |
| Express.js (Node)            | `proyecto-beex-fe`    | `blue`         |                     |
| **MERN (MongoDB + Express)** | `proyecto-mern`       | **`orange`**   | **← ESTE PROYECTO** |
| Next.js fullstack            | `proyecto-be-fe-next` | `violet`       |                     |
| Spring Boot Java             | `proyecto-besb-fe`    | `amber`        |                     |
| Spring Boot Kotlin           | `proyecto-besbk-fe`   | `fuchsia`      |                     |
| Go REST API                  | `proyecto-bego-fe`    | `cyan`         |                     |

#### Regla fundamental: usar `accent-*`, nunca colores concretos

Los componentes y páginas del frontend **NUNCA** deben usar clases de color concretas
para elementos de marca o interacción primaria. **SIEMPRE** usar el token `accent-*`:

```typescript
// ✅ CORRECTO — token semántico, funciona en todos los proyectos de la serie
<button className="bg-accent-600 hover:bg-accent-700 text-white ...">Enviar</button>
<a className="text-accent-600 hover:text-accent-700 dark:text-accent-400">Ver más</a>
<input className="focus:border-accent-500 focus:ring-accent-500/20 ..." />

// ❌ INCORRECTO — color concreto hardcodeado (rompe la portabilidad del tema)
<button className="bg-emerald-600 hover:bg-emerald-700 ...">Enviar</button>
<button className="bg-blue-600 hover:bg-blue-700 ...">Enviar</button>
```

#### El único lugar donde se define el color concreto: `fe/src/index.css`

```css
/* fe/src/index.css — @theme inline */
/* Para este proyecto: accent = orange */
--color-accent-600: var(--color-orange-600);

/* Para clonar hacia FastAPI: cambiar orange por emerald */
--color-accent-600: var(--color-emerald-600);
```

**Clonar el tema** = cambiar 11 líneas en `index.css` + 2 valores SVG en el logo.
Ver `docs/referencia-tecnica/design-system.md` para instrucciones completas.

#### Colores semánticos — NO cambian entre proyectos

Los colores de estado son fijos independientemente del stack. **No reemplazar con `accent-`**:

| Estado      | Color    | Ejemplo de uso                         |
| ----------- | -------- | -------------------------------------- |
| Éxito       | `green`  | Alert type="success"                   |
| Error       | `red`    | Alert type="error", inputs inválidos   |
| Información | `blue`   | Alert type="info", avisos informativos |
| Advertencia | `yellow` | Avisos no críticos                     |

```typescript
// ✅ CORRECTO — info SIEMPRE azul aunque el proyecto sea emerald/violet/amber
<Alert type="info" message="Tu sesión expirará en 5 minutos" />
// Internamente usa bg-blue-50 text-blue-800 — CORRECTO, no cambia
```

---

## 15. Reglas para Copilot / IA — Al Generar Código

1. **Dividir respuestas largas** — Si la implementación es extensa, dividirla en pasos incrementales. No generar todo de golpe.
2. **Codigo generado = código probado** — Siempre incluir o sugerir tests para lo que se genere.
3. **Comentarios pedagógicos** — Cada bloque significativo debe tener comentarios con ¿Qué? ¿Para qué? ¿Impacto?
4. **Tipos obligatorios** — Nunca omitir tipado TypeScript en BE ni en FE. Nunca usar `any`.
5. **Formato correcto** — Respetar ESLint + Prettier para todo el código (BE y FE).
6. **Usar las herramientas correctas** — `pnpm` para Node.js en ambos lados. Sin excepciones.
7. **Variables de entorno** — Toda configuración sensible va en `.env`, nunca hardcodeada.
8. **Conventional Commits** — Sugerir mensajes de commit con formato correcto.
9. **Seguridad primero** — Nunca almacenar passwords en texto plano, nunca exponer secrets.
10. **Legibilidad sobre cleverness** — El código debe ser entendible para un aprendiz.

---

## 16. Plan de Trabajo — Fases

> Cada fase es independiente y verificable. No avanzar a la siguiente sin completar y probar la actual.

### Fase 0 — Fundamentos y Configuración Base

- [ ] Crear `.github/copilot-instructions.md` (este archivo)
- [ ] Crear `.gitignore` raíz
- [ ] Crear `docker-compose.yml` con MongoDB 7 + Mailpit
- [ ] Crear `README.md` con descripción, stack, prerrequisitos y setup

### Fase 1 — Backend Setup

- [ ] Inicializar proyecto Node.js + TypeScript en `be/` con `pnpm init`
- [ ] Instalar dependencias de producción y desarrollo
- [ ] Configurar `tsconfig.json` y `eslint.config.js`
- [ ] Crear `src/config/env.ts` — validación de variables de entorno (envalid)
- [ ] Crear `src/db/connection.ts` — conexión a MongoDB via Mongoose
- [ ] Crear `src/app.ts` — Express app con middleware (cors, helmet, morgan)
- [ ] Crear `src/index.ts` — arranque del servidor
- [ ] Crear `.env.example` y `.env`
- [ ] ✅ Verificar: `pnpm dev` → servidor corriendo en `http://localhost:5000`

### Fase 2 — Modelos Mongoose

- [ ] Crear `src/models/User.ts` — schema e interfaz IUser
- [ ] Crear `src/models/PasswordResetToken.ts` — schema e interfaz
- [ ] ✅ Verificar: modelos se registran correctamente al conectar a MongoDB

### Fase 3 — Autenticación Backend

- [ ] Crear `src/utils/security.ts` — hashing (bcryptjs) y JWT (jsonwebtoken)
- [ ] Crear `src/utils/email.ts` — envío de emails (nodemailer)
- [ ] Crear `src/middleware/auth.middleware.ts` — verificación de JWT
- [ ] Crear `src/middleware/validate.middleware.ts` — runner de express-validator
- [ ] Crear `src/services/auth.service.ts` — lógica de negocio
- [ ] Crear `src/controllers/auth.controller.ts` — handlers HTTP
- [ ] Crear `src/controllers/users.controller.ts` — handler GET /me
- [ ] Crear `src/routes/auth.routes.ts` — endpoints de auth con validaciones
- [ ] Crear `src/routes/users.routes.ts` — endpoint de perfil
- [ ] Registrar routers en `src/app.ts`
- [ ] ✅ Verificar: probar todos los endpoints con curl o Postman/Insomnia

### Fase 4 — Tests Backend

- [ ] Configurar Jest + ts-jest + mongodb-memory-server
- [ ] Crear `src/__tests__/setup.ts` — configuración global (in-memory MongoDB)
- [ ] Crear `src/__tests__/auth.test.ts` — tests de integración con supertest
- [ ] ✅ Verificar: `pnpm test` → todos los tests pasan con ≥80% cobertura

### Fase 5 — Frontend Setup

- [ ] Inicializar proyecto Vite con React + TypeScript en `fe/`
- [ ] Instalar dependencias con `pnpm`
- [ ] Configurar TailwindCSS con tema `accent = orange`
- [ ] Configurar TypeScript strict mode
- [ ] Crear `.env.example`
- [ ] ✅ Verificar: `pnpm dev` → app base visible en `http://localhost:5173`

### Fase 6 — Frontend Auth

- [ ] Crear tipos TypeScript (`types/auth.ts`)
- [ ] Crear cliente HTTP (`api/auth.ts`)
- [ ] Crear AuthContext + Provider
- [ ] Crear hook `useAuth`
- [ ] Crear componentes UI (InputField, Button, Alert)
- [ ] Crear ProtectedRoute
- [ ] Crear páginas: Login, Register, Dashboard, ChangePassword, ForgotPassword, ResetPassword
- [ ] Crear `LandingPage.tsx` — página pública en ruta `/` con logo SVG, features, pasos, stack y CTAs
- [ ] Configurar rutas en App.tsx (ruta `/` muestra `<LandingPage />`, no redirect)
- [ ] Crear páginas legales: `TerminosDeUsoPage.tsx`, `PoliticaPrivacidadPage.tsx`, `PoliticaCookiesPage.tsx`
- [ ] Crear `LegalLayout.tsx` — layout compartido para páginas legales (secciones numeradas)
- [ ] Registrar rutas `/terminos-de-uso`, `/privacidad`, `/cookies` en App.tsx
- [ ] Agregar nav de aviso legal con 3 enlaces en el footer de `LandingPage.tsx`
- [ ] Crear `ContactPage.tsx` — formulario de contacto público con validación y envío simulado
- [ ] Registrar ruta `/contacto` en App.tsx
- [ ] Agregar enlace "Contacto" en nav de aviso legal del footer de `LandingPage.tsx`
- [ ] ✅ Verificar: flujo completo funciona contra la API

### Fase 7 — Tests Frontend

- [ ] Configurar Vitest + Testing Library
- [ ] Crear tests para componentes y flujos de auth
- [ ] ✅ Verificar: `pnpm test` → todos los tests pasan

### Fase 8 — Documentación Final

- [ ] Crear `docs/referencia-tecnica/architecture.md` — arquitectura y diagramas
- [ ] Crear `docs/referencia-tecnica/api-endpoints.md` — documentación de endpoints
- [ ] Crear `docs/referencia-tecnica/database-schema.md` — esquema de colecciones
- [ ] Crear `docs/referencia-tecnica/design-system.md` — sistema de temas
- [ ] Actualizar `README.md` con instrucciones finales
- [ ] ✅ Verificar: documentación completa y coherente

### Fase 9 — Internacionalización (i18n)

- [ ] Backend: agregar campo `locale` en schema User (ya incluido en modelo base)
- [ ] Backend: agregar endpoint `PATCH /api/v1/users/me/locale`
- [ ] Backend: agregar función `updateUserLocale` en `auth.service.ts`
- [ ] Backend: agregar test para endpoint de locale
- [ ] Frontend: instalar `react-i18next`, `i18next`, `i18next-browser-languagedetector`
- [ ] Frontend: crear `src/locales/es/translation.json` — catálogo español
- [ ] Frontend: crear `src/locales/en/translation.json` — catálogo inglés
- [ ] Frontend: crear `src/i18n.ts` — configuración de i18next
- [ ] Frontend: importar i18n en `main.tsx`
- [ ] Frontend: crear componente `LanguageSwitcher`
- [ ] Frontend: integrar `LanguageSwitcher` en `Navbar`
- [ ] Frontend: adaptar todas las páginas de auth al hook `useTranslation()`
- [ ] Frontend: sincronizar preferencia de idioma con backend (usuario autenticado)
- [ ] Frontend: agregar mock de i18n en `setup.ts` para tests
- [ ] Frontend: crear test para `LanguageSwitcher`
- [ ] ✅ Verificar: la app cambia de idioma completamente al seleccionar ES/EN

---

## 17. Verificación Final del Sistema

```bash
# 1. Levantar base de datos y Mailpit
docker compose up -d

# 2. Levantar backend
cd be && pnpm dev
# → API disponible en http://localhost:5000

# 3. Levantar frontend (en otra terminal)
cd fe && pnpm dev
# → App disponible en http://localhost:5173

# 4. Ejecutar tests backend
cd be && pnpm test

# 5. Ejecutar tests frontend
cd fe && pnpm test

# 6. Flujo manual completo:
#    Registro → Login → Ver perfil → Cambiar contraseña →
#    Logout → Forgot password → Reset password → Login con nueva contraseña
```

---

> **Recuerda**: La calidad no es una opción, es una obligación.
> Cada línea de código es una oportunidad de aprender y enseñar.
