# Esquema de Base de Datos — NN Auth System

<!--
  ¿Qué? Documentación del esquema documental de MongoDB para el sistema NN Auth System.
  ¿Para qué? Que cualquier desarrollador entienda la estructura de datos del sistema
             sin necesidad de leer los modelos Mongoose ni conectarse a la base de datos.
  ¿Impacto? El esquema es el contrato entre el backend y la base de datos.
             Entender las colecciones, campos, índices y relaciones es fundamental para
             escribir queries eficientes y modelos Mongoose correctos.
-->

> **Motor**: MongoDB 7 (Docker en desarrollo)
> **ODM**: Mongoose 8 — Object Document Mapper para Node.js
> **Migraciones**: No existen en MongoDB — los cambios de esquema se gestionan
> con estrategias de evolución de documentos (backward-compatible fields)

---

## Diagrama Entidad-Relación (ER)

```
┌──────────────────────────────────────────────┐
│                    users                     │
├──────────────────────────────────────────────┤
│ PK  _id               ObjectId               │
│     email             String  UNIQUE         │
│     firstName         String                 │
│     lastName          String                 │
│     hashedPassword    String                 │
│     isActive          Boolean = true         │
│     isEmailVerified   Boolean = false        │
│     locale            String  = "es"         │
│     createdAt         Date  (auto)           │
│     updatedAt         Date  (auto)           │
└────────────────────┬─────────────────────────┘
                     │ 1
                     │
                     │ N
                     ▼
┌──────────────────────────────────────────────┐
│            passwordresettokens               │
├──────────────────────────────────────────────┤
│ PK  _id        ObjectId                      │
│ FK  userId     ObjectId → users._id          │
│     token      String  UNIQUE                │
│     expiresAt  Date                          │
│     used       Boolean = false               │
│     createdAt  Date  (auto)                  │
└──────────────────────────────────────────────┘
```

**Relaciones**:

- Un `User` puede tener **muchos** `PasswordResetToken` (uno por cada solicitud de recuperación)
- `userId` es una referencia (`ref: 'User'`) — Mongoose puede popularlo con `.populate('userId')`
- No hay foreign key enforcement en MongoDB — la integridad la garantiza el código en los services

> **Nota pedagógica**: A diferencia de PostgreSQL, MongoDB no tiene relaciones declarativas
> con `ON DELETE CASCADE`. Si se borra un usuario, hay que eliminar sus tokens manualmente
> desde el service, o usar un middleware Mongoose `pre('deleteOne', ...)`.

---

## Colección: `users`

Almacena los datos de todos los usuarios registrados en el sistema.

### Schema Mongoose (`be/src/models/User.ts`)

```typescript
const userSchema = new Schema<IUser>(
  {
    email:           { type: String, required: true, unique: true, lowercase: true, trim: true },
    firstName:       { type: String, required: true, trim: true },
    lastName:        { type: String, required: true, trim: true },
    hashedPassword:  { type: String, required: true },
    isActive:        { type: Boolean, default: true },
    isEmailVerified: { type: Boolean, default: false },
    locale:          { type: String, enum: ['es', 'en'], default: 'es' },
  },
  { timestamps: true },   // genera createdAt y updatedAt automáticamente
);

// Índice explícito para búsquedas por email (login frecuente)
userSchema.index({ email: 1 }, { unique: true });
```

### Campos

| Campo             | Tipo     | Requerido | Default  | Descripción                                                      |
| ----------------- | -------- | --------- | -------- | ---------------------------------------------------------------- |
| `_id`             | ObjectId | Auto      | —        | PK generada por MongoDB — 12 bytes, incluye timestamp            |
| `email`           | String   | Sí        | —        | Credencial de login. `unique: true` + índice B-tree              |
| `firstName`       | String   | Sí        | —        | Nombre para mostrar en la interfaz                               |
| `lastName`        | String   | Sí        | —        | Apellido para mostrar en la interfaz                             |
| `hashedPassword`  | String   | Sí        | —        | Hash bcrypt de la contraseña. **Nunca texto plano**              |
| `isActive`        | Boolean  | No        | `true`   | Permite desactivar cuentas sin borrar datos (soft delete)        |
| `isEmailVerified` | Boolean  | No        | `false`  | Reservado para futura verificación de email                      |
| `locale`          | String   | No        | `"es"`   | Preferencia de idioma del usuario: `"es"` o `"en"`               |
| `createdAt`       | Date     | Auto      | `now()`  | Generado por `{ timestamps: true }` en Mongoose                  |
| `updatedAt`       | Date     | Auto      | `now()`  | Actualizado automáticamente en cada `save()`                     |

### Índices

| Índice               | Campo   | Tipo   | Razón                                                  |
| -------------------- | ------- | ------ | ------------------------------------------------------ |
| `_id_` (default)     | `_id`   | PK     | Búsquedas por ID (joins manuales con tokens)           |
| `email_1` (unique)   | `email` | UNIQUE | Login frecuente por email — O(log n) con índice B-tree |

### Notas de seguridad

- `hashedPassword` almacena el hash bcrypt (formato `$2b$12$...`) — nunca se devuelve en las responses.
  Excluir explícitamente con `select: false` o en el service con `.select('-hashedPassword')`.
- `isActive` permite bloquear cuentas sospechosas sin perder datos históricos.
- `locale` solo acepta `"es"` o `"en"` — Mongoose valida con `enum` y rechaza cualquier otro valor.

---

## Colección: `passwordresettokens`

Almacena tokens temporales para el flujo de recuperación de contraseña.

> **Nombre de la colección**: Mongoose pluraliza automáticamente el modelo `PasswordResetToken`
> a `passwordresettokens` (todo en minúsculas). Verificar con `mongoose.modelNames()`.

### Schema Mongoose (`be/src/models/PasswordResetToken.ts`)

```typescript
const passwordResetTokenSchema = new Schema<IPasswordResetToken>(
  {
    userId:    { type: Schema.Types.ObjectId, ref: 'User', required: true, index: true },
    token:     { type: String, required: true, unique: true },
    expiresAt: { type: Date, required: true },
    used:      { type: Boolean, default: false },
  },
  { timestamps: true },
);

passwordResetTokenSchema.index({ token: 1 }, { unique: true });
```

### Campos

| Campo       | Tipo     | Requerido | Default  | Descripción                                                    |
| ----------- | -------- | --------- | -------- | -------------------------------------------------------------- |
| `_id`       | ObjectId | Auto      | —        | PK generada por MongoDB                                        |
| `userId`    | ObjectId | Sí        | —        | Referencia a `users._id`. No hay CASCADE — se borra en service |
| `token`     | String   | Sí        | —        | Token UUID aleatorio enviado en el email de recuperación       |
| `expiresAt` | Date     | Sí        | —        | Expiración: **1 hora** desde la creación                       |
| `used`      | Boolean  | No        | `false`  | `true` una vez utilizado — previene reúso del enlace           |
| `createdAt` | Date     | Auto      | `now()`  | Generado por `{ timestamps: true }`                            |

### Índices

| Índice             | Campo    | Tipo   | Razón                                            |
| ------------------ | -------- | ------ | ------------------------------------------------ |
| `_id_` (default)   | `_id`    | PK     | —                                                |
| `token_1` (unique) | `token`  | UNIQUE | Reset password busca por token — requiere índice |
| `userId_1`         | `userId` | Normal | Buscar todos los tokens de un usuario            |

### Ciclo de vida de un token de reset

```
1. Usuario solicita reset → service crea documento (used: false, expiresAt: now + 1h)
2. Se envía email con enlace: {FRONTEND_URL}/reset-password?token=<uuid>
3. Usuario hace clic → backend ejecuta PasswordResetToken.findOne({ token }):
   - ¿Existe? → OK
   - ¿expiresAt > new Date()? → OK (no expirado)
   - ¿used === false? → OK (no reutilizado)
4. Se actualiza User.hashedPassword + se marca token.used = true
5. Cualquier intento posterior con el mismo token → rechazado (400)
```

---

## Convenciones de Nomenclatura en MongoDB/Mongoose

| Aspecto              | Convención                | Ejemplo                                      |
| -------------------- | ------------------------- | -------------------------------------------- |
| Colecciones          | `camelCase`, plural       | `users`, `passwordresettokens`               |
| Campos               | `camelCase`               | `hashedPassword`, `createdAt`, `firstName`   |
| Primary Keys         | `_id` (ObjectId)          | `_id: ObjectId("507f1f77bcf86cd799439011")`  |
| Foreign Keys / Refs  | `<entidad>Id`             | `userId: ObjectId` con `ref: 'User'`         |
| Timestamps           | `createdAt` / `updatedAt` | Generados por `{ timestamps: true }`         |
| Nombres de modelos   | `PascalCase`, singular    | `User`, `PasswordResetToken`                 |

---

## ¿Por qué ObjectId y no UUID?

MongoDB genera automáticamente un `_id` de tipo **ObjectId** (12 bytes):

```
ObjectId("507f1f77bcf86cd799439011")
        └──────────────────────────┘
         4 bytes timestamp + 5 bytes random + 3 bytes counter
```

**Ventajas del ObjectId vs Integer autoincremental:**

- **No predecible**: No se pueden adivinar IDs de otros usuarios
- **Timestamp integrado**: `_id.getTimestamp()` devuelve la fecha de creación sin campo extra
- **Distribuido**: Se puede generar en el cliente sin coordinación con el servidor

**vs UUID v4**: ObjectId es más compacto (12 vs 16 bytes) y lleva timestamp integrado.
Para este proyecto, ObjectId es la elección natural de MongoDB.

---

## Evolución de Esquemas (sin migraciones)

> **Nota pedagógica**: MongoDB es schemaless — no existen migraciones como en PostgreSQL/Alembic.
> Mongoose añade validación a nivel de aplicación, pero la BD acepta cualquier documento.

Estrategias para cambios de esquema en producción:

```
1. Agregar campo opcional (backward-compatible):
   - Añadir el campo en el schema Mongoose con un default
   - Los documentos viejos usan el default; los nuevos tienen el valor real
   - No se necesita tocar los documentos existentes

2. Renombrar campo:
   - Script de migración: db.users.updateMany({}, { $rename: { oldField: 'newField' } })

3. Cambiar tipo de campo:
   - Requiere script de migración explícito en ventana de mantenimiento

Ejemplo:
   cd be && node scripts/migrate-add-locale.js
```

---

## Ejemplos de Documentos en MongoDB

### Colección `users`

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "email": "carlos.garcia@ejemplo.com",
  "firstName": "Carlos",
  "lastName": "García",
  "hashedPassword": "$2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy",
  "isActive": true,
  "isEmailVerified": false,
  "locale": "es",
  "createdAt": "2026-04-22T10:00:00.000Z",
  "updatedAt": "2026-04-22T10:00:00.000Z"
}
```

### Colección `passwordresettokens`

```json
{
  "_id": "60c72b2f9b1d4c0015a1f3e2",
  "userId": "507f1f77bcf86cd799439011",
  "token": "a3f8c2d1-7e4b-4f9a-b2c6-1d3e5f7a9b0c",
  "expiresAt": "2026-04-22T11:00:00.000Z",
  "used": false,
  "createdAt": "2026-04-22T10:00:00.000Z",
  "updatedAt": "2026-04-22T10:00:00.000Z"
}
```
