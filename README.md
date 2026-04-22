# 🔐 NN Auth System

<!--
  ¿Qué? Documentación principal del proyecto NN Auth System.
  ¿Para qué? Guiar a cualquier desarrollador o aprendiz para entender, configurar y ejecutar el proyecto.
  ¿Impacto? Sin este README, los nuevos colaboradores no sabrían cómo levantar el proyecto
  ni entenderían su propósito, arquitectura o convenciones.
-->

> **Proyecto educativo** — SENA | Abril 2026

Sistema de autenticación completo para una empresa genérica **"NN"**, diseñado como ejercicio formativo.
Incluye landing page pública, registro de usuarios, login, cambio de contraseña y recuperación por email.

---

## 📋 Tabla de Contenidos

- [🔐 NN Auth System](#-nn-auth-system)
  - [📋 Tabla de Contenidos](#-tabla-de-contenidos)
  - [🛠️ Stack Tecnológico](#️-stack-tecnológico)
  - [✅ Prerrequisitos](#-prerrequisitos)
    - [Instalar pnpm (si no lo tienes)](#instalar-pnpm-si-no-lo-tienes)
  - [🚀 Instalación y Setup](#-instalación-y-setup)
    - [1. Clonar el repositorio](#1-clonar-el-repositorio)
    - [2. Levantar la base de datos](#2-levantar-la-base-de-datos)
    - [3. Configurar el Backend](#3-configurar-el-backend)
    - [4. Configurar el Frontend](#4-configurar-el-frontend)
  - [▶️ Ejecución](#️-ejecución)
    - [Levantar todo el sistema (3 terminales)](#levantar-todo-el-sistema-3-terminales)
  - [🧪 Testing](#-testing)
    - [Backend](#backend)
    - [Frontend](#frontend)
    - [Linting](#linting)
  - [📁 Estructura del Proyecto](#-estructura-del-proyecto)
  - [📏 Convenciones](#-convenciones)
  - [📚 Documentación Adicional](#-documentación-adicional)
  - [🎓 Propósito Educativo](#-propósito-educativo)
  - [⚠️ Exención de Responsabilidades](#️-exención-de-responsabilidades)
  - [📄 Licencia](#-licencia)

---

## 🛠️ Stack Tecnológico

| Capa              | Tecnologías                                                      |
| ----------------- | ---------------------------------------------------------------- |
| **Backend**       | Node.js 20 LTS, Express.js 4, TypeScript 5, Mongoose 8, JWT      |
| **Frontend**      | React 18+, Vite 6, TypeScript 5, TailwindCSS 4+                  |
| **Base de datos** | MongoDB 7 (Docker Compose)                                       |
| **Email (dev)**   | Mailpit — captura SMTP local, UI en puerto 8025                  |
| **Testing**       | jest + supertest + mongodb-memory-server (BE), Vitest + Testing Library (FE) |
| **Linting**       | ESLint + Prettier (BE y FE)                                      |

---

## ✅ Prerrequisitos

Antes de comenzar, asegúrate de tener instalado:

| Herramienta        | Versión mínima | Verificar con            |
| ------------------ | -------------- | ------------------------ |
| **Node.js**        | 20 LTS+        | `node --version`         |
| **pnpm**           | 9+             | `pnpm --version`         |
| **Docker**         | 24+            | `docker --version`       |
| **Docker Compose** | 2.20+          | `docker compose version` |
| **Git**            | 2.40+          | `git --version`          |

> ⚠️ **Importante**: Usar **pnpm** como gestor de paquetes de Node.js. **Nunca usar npm ni yarn.**

> 🖥️ **Usuarios de Windows** — Usa **Git Bash** o **WSL2** como terminal.
> Los comandos de este proyecto usan sintaxis Bash. **No uses CMD ni PowerShell.**

### Instalar pnpm (si no lo tienes)

```bash
# Opción recomendada — vía corepack (incluido con Node.js 16+)
corepack enable
corepack prepare pnpm@latest --activate

# Alternativa — instalación independiente
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

---

## 🚀 Instalación y Setup

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd proyecto
```

### 2. Levantar la base de datos

```bash
# Inicia MongoDB 7 + Mailpit (captura de emails) en contenedores Docker
docker compose up -d

# Verificar que están corriendo
docker compose ps
# Deberías ver nn_auth_mongo y nn_auth_mailpit con estado "running"
```

### 3. Configurar el Backend

```bash
cd be

# Instalar dependencias con pnpm (¡NUNCA con npm!)
pnpm install

# Copiar y configurar variables de entorno
cp .env.example .env
# Editar .env con tus valores si es necesario
```

### 4. Configurar el Frontend

```bash
cd fe

# Instalar dependencias con pnpm (¡NUNCA con npm!)
pnpm install

# Copiar y configurar variables de entorno
cp .env.example .env
```

---

## ▶️ Ejecución

### Levantar todo el sistema (3 terminales)

```bash
# Terminal 1 — Base de datos (si no está corriendo)
docker compose up -d

# Terminal 2 — Backend (Express.js)
cd be && pnpm dev
# → API disponible en http://localhost:5000

# Terminal 3 — Frontend (React + Vite)
cd fe && pnpm dev
# → Landing page en http://localhost:5173
# → App disponible en http://localhost:5173
```

> 📧 **Mailpit** — bandeja de entrada de emails de desarrollo: `http://localhost:8025`
> Aquí se capturan los emails de recuperación de contraseña.

---

## 🧪 Testing

### Backend

```bash
cd be

# Ejecutar todos los tests
pnpm test

# Ejecutar con cobertura
pnpm test:coverage

# Ejecutar un test específico
pnpm test -- --testPathPattern=auth
```

### Frontend

```bash
cd fe

# Ejecutar todos los tests
pnpm test

# Ejecutar en modo watch
pnpm test:watch

# Ejecutar con cobertura
pnpm test:coverage
```

### Linting

```bash
# Backend
cd be && pnpm lint && pnpm format

# Frontend
cd fe && pnpm lint && pnpm format
```

---

## 📁 Estructura del Proyecto

```
proyecto-mern/
├── .github/copilot-instructions.md   # Reglas y convenciones del proyecto
├── .gitignore                        # Archivos ignorados por git
├── docker-compose.yml                # MongoDB 7 + Mailpit para desarrollo
├── README.md                         # ← Este archivo
├── docs/                             # Documentación técnica
├── assets/                           # Recursos estáticos
├── be/                               # Backend — Express.js + TypeScript
│   ├── src/
│   │   ├── index.ts                  # Punto de entrada — inicia el servidor HTTP
│   │   ├── app.ts                    # Express app — middleware y rutas
│   │   ├── config/env.ts             # Variables de entorno validadas
│   │   ├── db/connection.ts          # Conexión a MongoDB via Mongoose
│   │   ├── models/                   # Modelos Mongoose (User, PasswordResetToken)
│   │   ├── routes/                   # Routers de Express (auth, users)
│   │   ├── controllers/              # Handlers HTTP
│   │   ├── services/                 # Lógica de negocio
│   │   ├── middleware/               # auth, validate
│   │   ├── utils/                    # security.ts, email.ts
│   │   └── __tests__/               # Tests con jest + supertest
│   ├── package.json                  # Dependencias (pnpm)
│   └── tsconfig.json                 # Configuración TypeScript
└── fe/                               # Frontend — React + Vite + TypeScript
    ├── src/
    │   ├── api/                      # Clientes HTTP
    │   ├── components/               # Componentes reutilizables
    │   ├── pages/                    # Páginas/vistas (Landing, Login, Register, Dashboard…)
    │   ├── hooks/                    # Custom hooks
    │   ├── context/                  # Context providers
    │   ├── locales/                  # Catálogos de traducción (es, en)
    │   └── types/                    # Tipos TypeScript
    ├── package.json                  # Dependencias (pnpm)
    └── vite.config.ts                # Configuración de Vite
```

---

## 📏 Convenciones

| Aspecto              | Regla                                            |
| -------------------- | ------------------------------------------------ |
| Nomenclatura técnica | Inglés (variables, funciones, clases, endpoints) |
| Comentarios/docs     | Español (con ¿Qué? ¿Para qué? ¿Impacto?)         |
| Commits              | Conventional Commits en inglés + What/For/Impact |
| TypeScript (BE y FE) | strict mode + ESLint + Prettier                  |
| Gestor de paquetes   | `pnpm` (BE y FE — nunca npm ni yarn)             |
| Testing              | Código generado = código probado                 |

Para las reglas completas, ver [`.github/copilot-instructions.md`](.github/copilot-instructions.md).

---

## 📚 Documentación Adicional

| Documento                                                                                  | Descripción                                              |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| [`docs/referencia-tecnica/architecture.md`](docs/referencia-tecnica/architecture.md)       | Arquitectura general, flujos y decisiones técnicas       |
| [`docs/referencia-tecnica/api-endpoints.md`](docs/referencia-tecnica/api-endpoints.md)     | Todos los endpoints con parámetros, respuestas y errores |
| [`docs/referencia-tecnica/database-schema.md`](docs/referencia-tecnica/database-schema.md) | Esquema de colecciones Mongoose                          |
| [`docs/referencia-tecnica/design-system.md`](docs/referencia-tecnica/design-system.md)     | Sistema de temas y tokens de color (accent = orange)     |
| [`docs/conceptos/owasp-top-10.md`](docs/conceptos/owasp-top-10.md)                         | Implementación del OWASP Top 10 2021                     |
| [`.github/copilot-instructions.md`](.github/copilot-instructions.md)                       | Reglas y convenciones del proyecto                       |

---

## 🎓 Propósito Educativo

Este proyecto está diseñado para **aprender haciendo**. Cada archivo, función y componente incluye comentarios pedagógicos que explican:

- **¿Qué?** — Qué hace este código
- **¿Para qué?** — Por qué existe y cuál es su propósito
- **¿Impacto?** — Qué pasa si no existiera o si se implementa mal

> _"La calidad no es una opción, es una obligación."_

---

## ⚠️ Exención de Responsabilidades

Este proyecto es de naturaleza **exclusivamente educativa**, desarrollado como ejercicio formativo en el marco del SENA.

- **No apto para producción** — El sistema no ha sido auditado ni endurecido para entornos productivos reales. No debe usarse para proteger datos sensibles de usuarios reales sin una revisión de seguridad profesional previa.
- **Credenciales de ejemplo** — Las contraseñas, claves secretas y cadenas de conexión presentes en `.env.example` y en la documentación son únicamente ilustrativas. **Nunca usarlas en producción.**
- **Sin garantía de disponibilidad** — El proyecto puede contener bugs o comportamientos no documentados propios de un entorno de aprendizaje.
- **Uso de terceros** — El proyecto referencia servicios externos (Resend, Neon, Supabase, Railway) como ejemplos pedagógicos. El autor no tiene afiliación con dichos servicios ni garantiza su disponibilidad o condiciones de uso.
- **Responsabilidad del aprendiz** — Cada aprendiz es responsable de comprender el código que ejecuta en su equipo y de no reutilizarlo sin entenderlo completamente.

> Este material se provee **"tal cual"**, sin garantías explícitas ni implícitas de ningún tipo.

---

## 📄 Licencia

[![CC BY-NC-SA 4.0](https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Este proyecto está licenciado bajo **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)**.

**Puedes:**

- ✅ Compartir — copiar y redistribuir el material en cualquier medio o formato
- ✅ Adaptar — remezclar, transformar y crear a partir del material (forks educativos permitidos)

**Bajo las siguientes condiciones:**

- 📝 **Atribución** — Debes dar crédito apropiado, enlazar la licencia e indicar si se realizaron cambios.
- 🚫 **No Comercial** — No puedes usar el material con fines comerciales.
- 🔄 **Compartir Igual** — Si remezclas o transformas el material, debes distribuir tus contribuciones bajo la misma licencia.

Consulta el archivo [LICENSE](./LICENSE) o visita [creativecommons.org/licenses/by-nc-sa/4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) para más información.