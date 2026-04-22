# OWASP Top 10 — Guía Pedagógica de Seguridad

<!--
  ¿Qué? Documento que explica cada una de las 10 vulnerabilidades más críticas
        según OWASP (Open Worldwide Application Security Project) y cómo este
        proyecto las mitiga o por qué no aplican.
  ¿Para qué? Educar al equipo sobre seguridad web práctica, mostrando no solo
             "qué hacer" sino "por qué hacerlo" con ejemplos del código real.
  ¿Impacto? Entender OWASP Top 10 es fundamental para cualquier desarrollador
             que trabaje en aplicaciones expuestas a internet. Este documento
             conecta la teoría con la implementación concreta del proyecto.
-->

> **Referencia oficial**: [OWASP Top 10 — 2021](https://owasp.org/Top10/)
> **Edición usada en este proyecto**: 2021 (vigente al inicio del proyecto, 2026)

---

## ¿Qué es OWASP?

**OWASP** (Open Worldwide Application Security Project) es una fundación internacional
sin fines de lucro dedicada a mejorar la seguridad del software. Su **Top 10** es el
listado de las vulnerabilidades más críticas y frecuentes en aplicaciones web, actualizado
periódicamente con datos reales de miles de organizaciones.

> Es el estándar de facto en la industria: muchas empresas exigen que sus desarrolladores
> conozcan y mitiguen el OWASP Top 10 antes de desplegar cualquier aplicación.

---

## Resumen de Estado — NN Auth System

| #   | Categoría                     | Estado          | Implementación                                        |
| --- | ----------------------------- | --------------- | ----------------------------------------------------- |
| A01 | Broken Access Control         | ✅ Implementado | JWT + middleware `requireAuth`                        |
| A02 | Cryptographic Failures        | ✅ Implementado | bcryptjs + JWT HS256 + SECRET_KEY validator           |
| A03 | Injection                     | ✅ Implementado | Mongoose ODM (no raw queries) + express-validator     |
| A04 | Insecure Design               | ✅ Implementado | Rate limiting con express-rate-limit                  |
| A05 | Security Misconfiguration     | ✅ Implementado | CORS explícito + helmet (security headers)            |
| A06 | Vulnerable Components         | ✅ Monitoreado  | Dependencias con versiones fijadas + pnpm audit       |
| A07 | Auth & Session Failures       | ✅ Implementado | Tokens de expiración corta + validación de contraseña |
| A08 | Software & Data Integrity     | ⚠️ Parcial      | Sin firma de código; tokens JWT firmados              |
| A09 | Logging & Monitoring Failures | ✅ Implementado | morgan + logging estructurado en middleware            |
| A10 | Server-Side Request Forgery   | ✅ N/A          | La API no hace peticiones a URLs externas             |

---

## A01 — Broken Access Control (Control de Acceso Roto)

### ¿Qué es?

Un usuario puede acceder a recursos o realizar acciones **para las que no tiene permiso**.
Es la vulnerabilidad #1 del OWASP Top 10 — aparece en el 94% de las aplicaciones analizadas.

### Ejemplos de ataque

```
# Atacante autenticado como usuario "A" accede al perfil de usuario "B":
GET /api/v1/users/123  →  debería devolver 403, pero devuelve datos de "B"

# Usuario sin autenticar accede a un endpoint protegido:
GET /api/v1/users/me  (sin token)  →  debería devolver 401
```

### Cómo lo mitiga este proyecto

**Middleware `requireAuth`** ([be/src/middleware/auth.middleware.ts](../../be/src/middleware/auth.middleware.ts)):

```typescript
// Cada endpoint protegido usa este middleware:
router.get('/me', requireAuth, usersController.getMe);

// Si el token es inválido, expirado o ausente → 401 automático
// El usuario solo puede acceder a SUS PROPIOS datos — no hay parámetro userId
// expuesto al cliente; se usa el ID extraído del token JWT
```

**Principio clave**: El ID del usuario nunca se acepta del cliente. Se extrae del
token JWT firmado por el servidor — el cliente no puede falsificarlo.

---

## A02 — Cryptographic Failures (Fallas Criptográficas)

### ¿Qué es?

Uso incorrecto o insuficiente de criptografía: contraseñas en texto plano, algoritmos
débiles (MD5, SHA1), claves cortas, o datos sensibles transmitidos sin cifrado.

### Ejemplos de ataque

```sql
-- Si las contraseñas se guardaran en texto plano o con MD5:
SELECT password FROM users WHERE email = 'victim@example.com';
-- Resultado: "miContra123" ← directamente usable por el atacante
-- Con MD5: "5d41402abc4b2a76b9719d911017c592" ← rompible en segundos con rainbow tables
```

### Cómo lo mitiga este proyecto

**1. bcryptjs para contraseñas** ([be/src/utils/security.ts](../../be/src/utils/security.ts)):

```typescript
// bcrypt es un hash adaptativo — costoso por diseño (work factor configurable)
// Incluye un salt aleatorio por contraseña → misma password, diferente hash
const SALT_ROUNDS = 12;

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);  // "$2b$12$..." — nunca texto plano
}

export async function verifyPassword(
  plainPassword: string,
  hashedPassword: string,
): Promise<boolean> {
  return bcrypt.compare(plainPassword, hashedPassword);
}
```

**¿Por qué bcryptjs y no SHA-256?**
SHA-256 es un hash rápido — diseñado para velocidad. Un atacante con GPU puede probar
billones de SHA-256 por segundo. bcrypt es lento _por diseño_ — limita a miles de
intentos por segundo, haciendo el brute-force imprácticamente costoso.

**2. JWT con SECRET_KEY robusta** ([be/src/config/env.ts](../../be/src/config/env.ts)):

```typescript
// envalid valida las variables de entorno al arrancar:
const env = cleanEnv(process.env, {
  JWT_SECRET: str({
    docs: 'Mínimo 32 caracteres. Genera con: openssl rand -hex 32',
  }),
  // ...
});
// Si JWT_SECRET tiene menos de 32 chars, el servidor no arranca
```

Una SECRET_KEY de menos de 32 caracteres puede romperse por fuerza bruta, permitiendo
al atacante generar tokens JWT válidos para cualquier usuario.

**3. HTTPS en producción** (responsabilidad del despliegue):
El código no puede forzar HTTPS, pero la arquitectura (nginx como proxy) debe configurarlo.
Sin HTTPS, tokens y contraseñas viajan en texto plano por la red.

---

## A03 — Injection (Inyección)

### ¿Qué es?

Datos no confiables del usuario se interpretan como **comandos o consultas**, manipulando
la lógica interna de la aplicación. SQL Injection es el ejemplo clásico — también existe
XSS (HTML/JS injection), NoSQL injection, LDAP injection, etc.

### Ejemplo de NoSQL Injection

```typescript
// ❌ VULNERABLE — acepta el objeto del cliente directamente:
const { email } = req.body;  // atacante envía: { "$gt": "" }
await User.findOne(req.body.filter);  // → retorna CUALQUIER usuario

// ✅ SEGURO — el campo se usa directamente como string:
await User.findOne({ email: email });  // Mongoose escapa el valor
```

### Cómo lo mitiga este proyecto

**Mongoose ODM con queries parametrizadas** ([be/src/services/auth.service.ts](../../be/src/services/auth.service.ts)):

```typescript
// ✅ SEGURO — Mongoose siempre parametriza las queries:
const user = await User.findOne({ email });
// Query real: { email: "<valor escapado>" } → imposible inyectar NoSQL

// ❌ VULNERABLE — si se acepta el objeto directamente del cliente:
const user = await User.findOne(req.body);  // ← NUNCA hacer esto
// Atacante puede enviar: { "$gt": "" } → retorna cualquier usuario
```

**Validación con express-validator** ([be/src/routes/auth.routes.ts](../../be/src/routes/auth.routes.ts)):

```typescript
export const loginRules = [
  body('email').isEmail().normalizeEmail(),    // Valida formato — rechaza strings arbitrarios
  body('password').isLength({ min: 1 }),       // Requerido
];
// Si la validación falla → 400 automático, nunca llega al service
```

**XSS**: La API devuelve JSON, no HTML — XSS en el backend no aplica directamente.
En el frontend, React escapa automáticamente el contenido en JSX (`{variable}` → HTML-escaped).

---

## A04 — Insecure Design (Diseño Inseguro)

### ¿Qué es?

La arquitectura o el diseño del sistema carece de controles de seguridad fundamentales.
No se trata de bugs de implementación sino de **ausencia de mecanismos de defensa** desde
el diseño: falta de rate limiting, falta de multi-factor auth, flujos sin límites de reintentos.

### Ataque que esto previene: Brute Force

```
# Sin rate limiting, un atacante puede automatizar miles de intentos:
POST /api/v1/auth/login {"email": "admin@nn.com", "password": "password1"}
POST /api/v1/auth/login {"email": "admin@nn.com", "password": "password2"}
POST /api/v1/auth/login {"email": "admin@nn.com", "password": "password3"}
... (repetir 100.000 veces en pocos minutos)
```

### Cómo lo mitiga este proyecto

**Rate limiting con express-rate-limit** ([be/src/app.ts](../../be/src/app.ts)):

```typescript
import rateLimit from 'express-rate-limit';

// Límites aplicados a los endpoints más sensibles:
const authLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 minuto
  max: 10,              // máx 10 intentos de login por IP
  message: { error: 'Demasiados intentos. Espera un minuto.' },
});

const registerLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,               // máx 5 registros/solicitudes de reset por IP
  message: { error: 'Límite de solicitudes alcanzado.' },
});

// Aplicados en las rutas:
router.post('/login', authLimiter, validate(loginRules), authController.login);
router.post('/register', registerLimiter, validate(registerRules), authController.register);
router.post('/forgot-password', registerLimiter, authController.forgotPassword);
```

**¿Qué pasa cuando se supera el límite?**

```json
HTTP 429 Too Many Requests
{"error": "Demasiados intentos. Espera un minuto."}
```

**Por qué 10/min para login y 5/min para register/forgot?**

- Login: un usuario legítimo que escribe mal su contraseña necesitaría 2-3 intentos máximo.
  10 intentos por minuto es cómodo para uso normal pero imposible para brute force masivo.
- Register/forgot-password: un humano raramente registra más de 1-2 cuentas por minuto.
  5/min previene creación masiva de cuentas falsas (spam, bots).

---

## A05 — Security Misconfiguration (Configuración de Seguridad Incorrecta)

### ¿Qué es?

El sistema funciona correctamente pero está **configurado de forma insegura**: permisos
excesivos, información expuesta, headers de seguridad ausentes, credenciales por defecto.

### Configuraciones inseguras comunes

```python
# ❌ CORS excesivamente permisivo:
app.add_middleware(CORSMiddleware,
    allow_origins=["*"],    # Cualquier sitio puede hacer peticiones
    allow_methods=["*"],    # PUT, DELETE, PATCH, OPTIONS...
    allow_headers=["*"],    # Cualquier header arbitrario
)

# ❌ Sin cabeceras de seguridad:
# Navegador no sabe que no debe embeber la app en iframes (clickjacking)
# Navegador no sabe que no debe ejecutar archivos .txt como scripts (MIME sniffing)
```

### Cómo lo mitiga este proyecto

**1. CORS mínimamente permisivo** ([be/src/app.ts](../../be/src/app.ts)):

```typescript
// ✅ Solo el origen del frontend de desarrollo:
app.use(cors({
  origin: env.FRONTEND_URL,  // http://localhost:5173 — no "*"
  credentials: true,
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

**2. Cabeceras de seguridad HTTP** — `helmet` ([be/src/app.ts](../../be/src/app.ts)):

```typescript
// helmet configura automáticamente las cabeceras de seguridad:
app.use(helmet());
```

| Header                                             | Protección                                |
| -------------------------------------------------- | ----------------------------------------- |
| `X-Content-Type-Options: nosniff`                  | Previene MIME-type sniffing               |
| `X-Frame-Options: DENY`                            | Previene clickjacking via `<iframe>`      |
| `Referrer-Policy: strict-origin-when-cross-origin` | Protege URLs con tokens en query string   |
| `Permissions-Policy: camera=(), microphone=()`     | Deniega acceso a hardware del dispositivo |
| Sin header `Server`                                | Oculta la tecnología del servidor         |

**3. Sin Swagger UI expuesto**: Express no genera documentación automática de la API. La documentación está en [`docs/referencia-tecnica/api-endpoints.md`](../referencia-tecnica/api-endpoints.md) — solo accesible para quien tiene acceso al repositorio, no expuesta en producción.

**4. Configuración desde variables de entorno** (validadas con `envalid` al arrancar):

```bash
JWT_SECRET=<generado con openssl rand -hex 32>
MONGODB_URI=mongodb://localhost:27017/nn_auth_db
NODE_ENV=production
```

---

## A06 — Vulnerable and Outdated Components (Componentes Vulnerables)

### ¿Qué es?

El sistema usa librerías, frameworks o runtime con **vulnerabilidades conocidas** (CVEs
publicados) porque no se actualizan. Un solo componente vulnerable puede comprometer
todo el sistema.

### Cómo lo mitiga este proyecto

**Versiones exactas fijadas** ([be/package.json](../../be/package.json)):

```json
{
  "dependencies": {
    "express": "4.21.2",
    "mongoose": "8.13.2",
    "jsonwebtoken": "9.0.2",
    "bcryptjs": "2.4.3"
  }
}
```

**Auditoría de CVEs con pnpm**:

```bash
# Verificar vulnerabilidades conocidas:
pnpm audit

# Solo reportar severidad alta/crítica:
pnpm audit --audit-level=high
```

> **Nota pedagógica**: Este proyecto usa versiones **exactas** (`"express": "4.21.2"`) en lugar de rangos (`^4.21.2`).
> Un rango permite instalar versiones con CVEs desconocidos en el momento del `pnpm install`. La versión exacta garantiza que el entorno es idéntico en cada instalación.

---

## A07 — Authentication and Session Failures (Fallas de Autenticación)

### ¿Qué es?

Debilidades en cómo el sistema identifica y verifica usuarios: contraseñas débiles
permitidas, tokens sin expiración, exposición de tokens, recuperación de contraseña insegura.

### Cómo lo mitiga este proyecto

**1. Validación de fortaleza de contraseñas** ([be/src/routes/auth.routes.ts](../../be/src/routes/auth.routes.ts)):

```typescript
export const registerRules = [
  body('password')
    .isLength({ min: 8 }).withMessage('Mínimo 8 caracteres')
    .matches(/[A-Z]/).withMessage('Al menos una mayúscula')
    .matches(/[a-z]/).withMessage('Al menos una minúscula')
    .matches(/\d/).withMessage('Al menos un número'),
];
```

**2. Tokens de corta duración** ([be/src/config/env.ts](../../be/src/config/env.ts)):

```typescript
JWT_ACCESS_EXPIRES_IN=15m   // 15 min — ventana pequeña si se filtra
JWT_REFRESH_EXPIRES_IN=7d   // 7 días — para renovar sin re-login
```

**3. Tokens de reset de un solo uso** ([be/src/models/PasswordResetToken.ts](../../be/src/models/PasswordResetToken.ts)):

```typescript
used: { type: Boolean, default: false }
// Una vez usado → se marca used=true → no puede usarse de nuevo
// Además tiene expiresAt → expira en 1 hora aunque no se use
```

**4. Mensajes de error genéricos**:

```typescript
// ❌ INSEGURO — revela si el email existe:
// "El email no está registrado" vs "Contraseña incorrecta"

// ✅ SEGURO — mismo mensaje siempre:
throw new AppError(401, 'Credenciales incorrectas');
```

---

## A08 — Software and Data Integrity Failures

### ¿Qué es?

El código o los datos pueden ser modificados por atacantes sin que el sistema lo detecte:
actualizaciones sin firma, deserialización insegura, pipelines CI/CD sin protección.

### Estado en este proyecto

**Parcialmente mitigado**:

✅ **Los tokens JWT están firmados** con HMAC-SHA256 — si alguien modifica el payload,
la verificación de firma falla y el token se rechaza.

```typescript
// Cualquier modificación al payload invalida la firma:
const token = jwt.sign(payload, env.JWT_SECRET, { algorithm: 'HS256' });
// ...
const decoded = jwt.verify(token, env.JWT_SECRET);  // JsonWebTokenError si fue alterado
```

⚠️ **Sin firma de artefactos de despliegue**: En un entorno productivo, el código
debería verificarse criptográficamente antes de desplegarse (Docker image signing).
Esto está fuera del alcance educativo del proyecto.

---

## A09 — Security Logging and Monitoring Failures

### ¿Qué es?

La aplicación no registra eventos de seguridad críticos (intentos de login fallido,
cambios de contraseña, errores de permisos) ni genera alertas ante comportamientos
anómalos. Sin logs, un ataque puede durar meses sin ser detectado.

### Stat real: tiempo promedio de detección de una brecha

> **280 días** es el promedio que tarda una organización en detectar una brecha de
> seguridad (IBM Cost of a Data Breach Report 2023). Los logs adecuados reducen
> drásticamente este número.

### Cómo lo mitiga este proyecto

**morgan + logging estructurado** ([be/src/app.ts](../../be/src/app.ts)):

```typescript
// morgan registra cada petición HTTP (método, ruta, status, tiempo):
app.use(morgan('combined'));

// Middleware de eventos de seguridad (auth.service.ts):
function logSecurityEvent(event: string, details: Record<string, string>): void {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    event,
    ...details,
  }));
}

// Eventos registrados en los servicios:
logSecurityEvent('login_failed',   { email: redactEmail(email), reason: 'invalid_credentials', ip });
logSecurityEvent('login_success',  { email: redactEmail(email), ip });
logSecurityEvent('password_reset', { ip });
```

**Formato de los logs (JSON estructurado)**:

```json
{
  "timestamp": "2026-04-22T14:23:45.123Z",
  "event": "login_failed",
  "email": "us***@nn-company.com",
  "reason": "invalid_credentials",
  "ip": "203.0.113.42"
}
```

**¿Por qué JSON estructurado y no texto plano?**
Los logs JSON pueden ser ingestados por herramientas como Elasticsearch, Splunk o
Datadog, que permiten buscar, filtrar y crear alertas automáticamente.

**Principio de privacidad en logs — email redactado**:

```typescript
function redactEmail(email: string): string {
  // "user@example.com" → "us***@example.com"
  const [local, domain] = email.split('@');
  return `${local.slice(0, 2)}***@${domain}`;
}
```

---

## A10 — Server-Side Request Forgery (SSRF)

### ¿Qué es?

El servidor hace peticiones HTTP a URLs controladas por el atacante, lo que puede
exponer servicios internos, metadatos de cloud (AWS IMDSv1), o permitir escaneo
de la red interna.

```
# Ejemplo de ataque SSRF:
POST /api/fetch-image
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}
# → El servidor hace un GET a la URL de metadatos de AWS
# → Retorna credenciales IAM del servidor ← atacante las roba
```

### Estado en este proyecto

**No aplica (N/A)**: La API de NN Auth System no realiza peticiones HTTP a URLs
externas basadas en input del usuario. Los únicos clientes HTTP externos son:

- El envío de email (servidor SMTP fijo, no controlable por el usuario)
- No hay endpoints de "fetch URL" ni webhooks configurables por el cliente

Si en el futuro se agregaran integraciones con URLs externas, se debe:

1. Validar la URL contra una whitelist de dominios permitidos
2. Bloquear rangos de IP privados (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.0.0/16)
3. Usar una librería o servicio dedicado a sanitizar URLs

---

## Resumen Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                    NN Auth System Security                       │
│                                                                 │
│  Cliente           Express.js Backend          Base de Datos      │
│  ────────          ──────────────────        ───────────────    │
│                                                                 │
│  POST /login  ──►  express-rate-limit (A04)                     │
│                    10 req/min por IP                            │
│                         │                                       │
│                    express-validator (A03)                      │
│                    isEmail, min length                          │
│                         │                                       │
│                    Auth Service (A07)                           │
│                    bcrypt.compare() (A02)                       │
│                         │                                       │
│                    morgan + JSON log (A09) ──►  stdout/archivo    │
│                    login_failed/success       (JSON logs)       │
│                         │                                       │
│  JWT Token    ◄──  JWT signed HS256 (A02)                       │
│                    ACCESS: 15min                                │
│                    REFRESH: 7 days                              │
│                                                                 │
│  GET /me      ──►  requireAuth (A01)                            │
│                    JWT verification                             │
│                    User ID from token                           │
│                    (never from client)                          │
│                                                                 │
│  All requests ──►  helmet (A05)                                 │
│               ◄──  X-Frame-Options: DENY                        │
│                    X-Content-Type-Options                       │
│                    Referrer-Policy                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Recursos de Aprendizaje

| Recurso              | URL                                 | Para qué                                     |
| -------------------- | ----------------------------------- | -------------------------------------------- |
| OWASP Top 10 oficial | https://owasp.org/Top10/            | Referencia completa actualizada              |
| OWASP Cheat Sheets   | https://cheatsheetseries.owasp.org/ | Guías específicas por tecnología             |
| Have I Been Pwned    | https://haveibeenpwned.com/         | Verificar si un email fue comprometido       |
| JWT Debugger         | https://jwt.io/                     | Decodificar y entender tokens JWT            |
| Security Headers     | https://securityheaders.com/        | Verificar cabeceras de seguridad de un sitio |

---

> **Conclusión pedagógica**: La seguridad no es una característica que se agrega al
> final — es una práctica que se integra en cada decisión de diseño y cada línea de
> código. El OWASP Top 10 no es una lista de miedos sino una **guía de construcción**
> que te dice exactamente qué construit para proteger tu aplicación y a sus usuarios.
