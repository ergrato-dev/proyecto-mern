# API Endpoints — NN Auth System

<!--
  ¿Qué? Documentación de referencia de todos los endpoints de la API REST del backend.
  ¿Para qué? Que cualquier desarrollador (frontend, backend, QA) pueda integrar o probar
             la API sin necesidad de leer el código fuente.
  ¿Impacto? Express.js no genera documentación automática — este archivo es la única
             referencia de la API para todo el equipo.
-->

> **Base URL**: `http://localhost:5000`
> **Versionamiento**: Todos los endpoints usan el prefijo `/api/v1/`
> **Formato**: JSON en request y response (Content-Type: `application/json`)
> **Autenticación**: JWT Bearer Token — `Authorization: Bearer <access_token>`

---

## Resumen de Endpoints

| Método | Ruta                           | Descripción                              | Auth | Rate Limit |
| ------ | ------------------------------ | ---------------------------------------- | ---- | ---------- |
| POST   | `/api/v1/auth/register`        | Registrar nuevo usuario                  | No   | 5/min      |
| POST   | `/api/v1/auth/login`           | Iniciar sesión, obtener tokens JWT       | No   | 10/min     |
| POST   | `/api/v1/auth/refresh`         | Renovar access token con refresh token   | No † | —          |
| POST   | `/api/v1/auth/change-password` | Cambiar contraseña (usuario autenticado) | Sí   | —          |
| POST   | `/api/v1/auth/forgot-password` | Solicitar email de recuperación          | No   | 5/min      |
| POST   | `/api/v1/auth/reset-password`  | Restablecer contraseña con token         | No † | —          |
| GET    | `/api/v1/users/me`             | Obtener perfil del usuario actual        | Sí   | —          |

> **†** No requiere `Authorization` header, pero sí un token específico en el body (refresh token o reset token).

---

## Códigos de Estado HTTP Comunes

| Código | Significado                                                         |
| ------ | ------------------------------------------------------------------- |
| 200    | OK — Operación exitosa                                              |
| 201    | Created — Recurso creado exitosamente (registro de usuario)         |
| 400    | Bad Request — Datos inválidos (email duplicado, validación fallida) |
| 401    | Unauthorized — Token inválido, expirado o ausente                   |
| 429    | Too Many Requests — Rate limit superado                             |
| 500    | Internal Server Error — Error no controlado del servidor            |

---

## Autenticación — Endpoints Auth

### POST /api/v1/auth/register

Registra un nuevo usuario en el sistema y envía un email de verificación.

**Rate limit**: 5 peticiones/minuto por IP

**Request body:**

```json
{
  "email": "usuario@ejemplo.com",
  "firstName": "Juan",
  "lastName": "Pérez",
  "password": "MiContrasena1"
}
```

| Campo       | Tipo   | Requerido | Validación                                   |
| ----------- | ------ | --------- | -------------------------------------------- |
| `email`     | string | Sí        | Formato email válido                          |
| `firstName` | string | Sí        | Mínimo 1 caracter (sin espacios en blanco)   |
| `lastName`  | string | Sí        | Mínimo 1 caracter (sin espacios en blanco)   |
| `password`  | string | Sí        | ≥8 chars, 1 mayúscula, 1 minúscula, 1 número |

**Respuesta exitosa (201 Created):**

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "email": "usuario@ejemplo.com",
  "firstName": "Juan",
  "lastName": "Pérez",
  "isActive": true,
  "isEmailVerified": false,
  "locale": "es",
  "createdAt": "2026-04-22T10:30:00.000Z",
  "updatedAt": "2026-04-22T10:30:00.000Z"
}
```

**Errores posibles:**

| Código | Condición                             |
| ------ | ------------------------------------- |
| 400    | Email ya registrado en el sistema     |
| 400    | Validación fallida (password débil)   |
| 429    | Rate limit superado (5/min)           |

---

### POST /api/v1/auth/login

Autentica al usuario y retorna un par de tokens JWT.

**Rate limit**: 10 peticiones/minuto por IP

**Request body:**

```json
{
  "email": "usuario@ejemplo.com",
  "password": "MiContrasena1"
}
```

| Campo      | Tipo   | Requerido |
| ---------- | ------ | --------- |
| `email`    | string | Sí        |
| `password` | string | Sí        |

**Respuesta exitosa (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

| Campo           | Tipo   | Descripción                         |
| --------------- | ------ | ----------------------------------- |
| `access_token`  | string | JWT de acceso — duración 15 min     |
| `refresh_token` | string | JWT de renovación — duración 7 días |
| `token_type`    | string | Siempre `"bearer"`                  |

**Cómo usar los tokens:**

```http
# Usar access_token en cada petición protegida:
GET /api/v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Errores posibles:**

| Código | Condición                                   |
| ------ | ------------------------------------------- |
| 401    | Email no registrado o contraseña incorrecta |
| 429    | Rate limit superado (10/min)                |

> **Nota de seguridad**: La API devuelve el mismo `401` tanto si el email no existe como si la contraseña es incorrecta. Esto evita la **enumeración de usuarios** — un atacante no puede saber si un email está registrado.

---

### POST /api/v1/auth/refresh

Genera un nuevo par de tokens usando un refresh token válido (rotación de tokens).

**Request body:**

```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Respuesta exitosa (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

> **Rotación de tokens**: El refresh token anterior queda invalidado al usar este endpoint. Siempre hay que guardar el nuevo par.

**Cuándo usar este endpoint:**

```
1. El frontend hace GET /api/v1/users/me → 401 (access_token expirado)
2. El frontend llama POST /api/v1/auth/refresh con el refresh_token guardado
3. El servidor retorna nuevos access_token + refresh_token
4. El frontend reintenta GET /api/v1/users/me con el nuevo access_token
```

**Errores posibles:**

| Código | Condición                                   |
| ------ | ------------------------------------------- |
| 401    | Refresh token inválido, expirado o ya usado |

---

### POST /api/v1/auth/change-password

Cambia la contraseña del usuario autenticado, verificando primero la contraseña actual.

**Auth requerida**: `Authorization: Bearer <access_token>`

**Request body:**

```json
{
  "currentPassword": "MiContrasenaActual1",
  "newPassword": "MiNuevaContrasena1"
}
```

| Campo             | Tipo   | Validación                                   |
| ----------------- | ------ | -------------------------------------------- |
| `currentPassword` | string | Contraseña actual correcta                   |
| `newPassword`     | string | ≥8 chars, 1 mayúscula, 1 minúscula, 1 número |

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Contraseña actualizada exitosamente"
}
```

**Errores posibles:**

| Código | Condición                                          |
| ------ | -------------------------------------------------- |
| 400    | La contraseña actual no es correcta                |
| 400    | Nueva contraseña no cumple requisitos de fortaleza |
| 401    | Token ausente, inválido o expirado                 |

---

### POST /api/v1/auth/forgot-password

Solicita el envío de un email con enlace de recuperación de contraseña.

**Rate limit**: 5 peticiones/minuto por IP

**Request body:**

```json
{
  "email": "usuario@ejemplo.com"
}
```

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Si el email está registrado, recibirás un enlace de recuperación"
}
```

> **Nota de seguridad**: La respuesta es **siempre idéntica**, sin importar si el email está registrado o no. Esto previene la **enumeración de usuarios** — un atacante no puede saber si un email existe consultando la respuesta.

El enlace enviado al email tiene este formato:

```
{FRONTEND_URL}/reset-password?token=<reset-token>
```

El token expira en **1 hora**.

**Errores posibles:**

| Código | Condición                   |
| ------ | --------------------------- |
| 400    | Email con formato inválido  |
| 429    | Rate limit superado (5/min) |

---

### POST /api/v1/auth/reset-password

Restablece la contraseña usando el token recibido por email.

**Request body:**

```json
{
  "token": "507f191e810c19729de860ea",
  "newPassword": "MiNuevaContrasena1"
}
```

| Campo         | Tipo   | Descripción                                |
| ------------- | ------ | ------------------------------------------ |
| `token`       | string | Token del email de recuperación            |
| `newPassword` | string | ≥8 chars, 1 mayúscula, 1 minúscula, 1 núm. |

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Contraseña restablecida exitosamente"
}
```

**Errores posibles:**

| Código | Condición                                          |
| ------ | -------------------------------------------------- |
| 400    | Token inválido, expirado o ya usado                |
| 400    | Nueva contraseña no cumple requisitos de fortaleza |

---

## Perfil de Usuario — Endpoints Users

### GET /api/v1/users/me

Retorna el perfil del usuario autenticado actualmente.

**Auth requerida**: `Authorization: Bearer <access_token>`

**Respuesta exitosa (200 OK):**

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "email": "usuario@ejemplo.com",
  "firstName": "Juan",
  "lastName": "Pérez",
  "isActive": true,
  "isEmailVerified": true,
  "locale": "es",
  "createdAt": "2026-04-22T10:30:00.000Z",
  "updatedAt": "2026-04-22T10:30:00.000Z"
}
```

| Campo             | Tipo             | Descripción                     |
| ----------------- | ---------------- | ------------------------------- |
| `_id`             | string (ObjectId)| Identificador único MongoDB      |
| `email`           | string           | Email del usuario               |
| `firstName`       | string           | Nombre                          |
| `lastName`        | string           | Apellido                        |
| `isActive`        | boolean          | Si la cuenta está activa        |
| `isEmailVerified` | boolean          | Si el email fue verificado      |
| `locale`          | string           | Idioma preferido (`es` / `en`)  |
| `createdAt`       | string (ISO8601) | Fecha de registro               |
| `updatedAt`       | string (ISO8601) | Fecha de última actualización   |

> **Nota de seguridad**: El campo `hashedPassword` **nunca** aparece en ninguna respuesta de la API. El schema Mongoose excluye explícitamente ese campo usando `select: false`.

**Errores posibles:**

| Código | Condición                          |
| ------ | ---------------------------------- |
| 401    | Token ausente, inválido o expirado |

---

## Rate Limiting — Detalles Técnicos

El rate limiting está implementado con [express-rate-limit](https://github.com/express-rate-limit/express-rate-limit), usando la IP del cliente como identificador.

| Endpoint                     | Límite | Razón                                      |
| ---------------------------- | ------ | ------------------------------------------ |
| `POST /auth/register`        | 5/min  | Prevenir creación masiva de cuentas falsas |
| `POST /auth/login`           | 10/min | Prevenir brute force de contraseñas        |
| `POST /auth/forgot-password` | 5/min  | Prevenir spam de emails de recuperación    |

**Respuesta cuando se supera el límite (429):**

```json
{
  "error": "Demasiados intentos. Espera un minuto."
}
```

---

## Flujos de Uso — Ejemplos Paso a Paso

### Flujo completo de registro

```
1. POST /api/v1/auth/register → 201 (cuenta creada)
2. POST /api/v1/auth/login { email, password } → 200 (access_token, refresh_token)
3. GET /api/v1/users/me con Bearer token → 200 (perfil completo)
```

### Flujo de renovación de sesión

```
1. (access_token expire a los 15 minutos)
2. POST /api/v1/auth/refresh { refresh_token } → 200 (nuevos tokens)
3. (Si refresh_token también expiró por >7 días)
   → 401 → Redirigir al login
```

### Flujo de recuperación de contraseña

```
1. POST /api/v1/auth/forgot-password { email } → 200 (siempre)
2. (El usuario hace clic en el enlace del email)
   Frontend captura: /reset-password?token=<uuid>
3. POST /api/v1/auth/reset-password { token, newPassword } → 200
4. POST /api/v1/auth/login { email, newPassword } → 200 (nuevos tokens)
```
