# Despliegue en Vercel, Render y Railway
esto es un cambio de Rafael Pablo Garcia
esto es un cambio

Este repositorio es una guía práctica para aprender a estructurar, dockerizar y desplegar una aplicación web siguiendo los principios de **CI/CD** con **GitHub Actions**, **Vercel**, **Render** y **Railway**.

Para el  **entorno de desarrollo local**, utilizaremos **Docker Compose** para levantar el frontend (Vue 3), el backend (FastAPI) y la base de datos en contenedores separados, emulando el entorno de producción.

Para el **despliegue en producción**, utilizaremos una plataforma especializada para cada servicio: **Vercel** para el frontend (Vue 3), **Render** para el backend (FastAPI) y **Railway** para la base de datos MySQL.

A modo de resumen, esta serían las tecnologías utilizadas:

- **Frontend:** Vue 3 + Vite + Tailwind CSS.
- **Backend:** FastAPI (Python).
- **Base de Datos:** MySQL.
- **Desarrollo:** Docker Compose.
- **Despliegue:** GitHub Actions + Render + Vercel + Railway.

<!-- --- -->

## Índice

0. [Introducción](#introducción)
1. [Estructura del Proyecto](#1-estructura-del-proyecto)
2. [Desarrollo local con Docker](#2-desarrollo-local-con-docker)
3. [Estructura del Frontend](#3-estructura-del-frontend)
4. [Estructura del Backend](#4-estructura-del-backend)
5. [Guía de Despliegue (CI/CD)](#5-guía-de-despliegue-cicd)
   - [Despliegue del Backend (Render)](#a-despliegue-del-backend-render)
   - [Despliegue del Frontend (Vercel)](#b-despliegue-del-frontend-vercel)
6. [Conceptos clave](#6-conceptos-clave)
7. [Tecnologías utilizadas](#7-tecnologías-utilizadas)
8. [Comandos útiles durante el desarrollo](#8-comandos-útiles-durante-el-desarrollo)
9. [Próximos pasos](#9-próximos-pasos)
10. [Licencia](#licencia)

<!-- --- -->

## 0. Introducción

### ¿Qué es Vercel?

[Vercel](https://vercel.com) es una plataforma de despliegue para frontend optimizada para Vue, React, Next.js, etc. Ofrece despliegue automático, CDN global y baja latencia.

### ¿Qué es Render?

[Render](https://render.com) es una plataforma de despliegue para backend, APIs y bases de datos. Maneja automáticamente SSL, escalado e infraestructura.

### ¿Qué es Railway?

[Railway](https://railway.com) es una plataforma de despliegue para bases de datos y servicios backend. Es ideal para gestionar bases de datos MySQL, PostgreSQL, etc.


<!-- --- -->

## 1. Estructura del Proyecto

El repositorio está organizado siguiendo el **patrón de monorepositorio sencillo**, donde cada servicio tiene su propia responsabilidad y configuración:

```text
.
├── backend/                # API REST con FastAPI (Python)
│   ├── main.py             # Lógica de la API y configuración de CORS
│   ├── requirements.txt    # Dependencias del proyecto
│   └── Dockerfile          # Imagen optimizada para producción (Multi-stage)
├── frontend/               # Aplicación Web con Vue 3 + Vite
│   ├── src/
│   │   ├── App.vue         # Componente principal
│   │   ├── main.ts         # Punto de entrada
│   │   ├── style.css       # Estilos globales con Tailwind
│   │   └── services/
│   │       └── api.ts      # Servicio para consumir API del backend
│   ├── index.html          # Punto de entrada HTML
│   ├── Dockerfile          # Imagen optimizada (Multi-stage)
│   ├── vite.config.ts      # Configuración de Vite
│   └── tailwind.config.js  # Configuración de Tailwind CSS
├── .github/workflows/      # Automatización (CI/CD)
│   ├── deploy-backend.yaml # Despliegue automático a Render
│   └── deploy-frontend.yaml# Despliegue automático a Vercel
├── .env.example            # Variables de entorno de ejemplo
├── compose.yaml            # Orquestación para desarrollo local
└── README.md               # Esta documentación
```

<!-- --- -->

## 2. Desarrollo local con Docker

Para asegurar que todos los desarrolladores trabajen en el mismo entorno, utilizamos **Docker Compose**. Esto nos permite emular cómo funcionará la aplicación en producción.

### Requisitos:

- Docker y Docker Compose instalados en la máquina de desarrollo.

### Pasos:

1. Clona el repositorio.
2. Crea tu archivo de entorno local:
   ```bash
   cp .env.example .env
   ```
3. Desde la raíz, levanta los servicios:
   ```bash
   docker compose up --build
   ```
4. Accede a las aplicaciones:
   - **Frontend:** [http://localhost:3000](http://localhost:3000)
   - **Backend API:** [http://localhost:8000](http://localhost:8000)
   - **Documentación Interactiva (Swagger):** [http://localhost:8000/docs](http://localhost:8000/docs)

<!-- --- -->

## 3. Estructura del Frontend

### Componentes principales:

- **App.vue:** Componente raíz que consume datos del backend
- **services/api.ts:** Servicio centralizador para peticiones HTTP con axios
- **style.css:** Estilos globales usando Tailwind CSS

### Consumiendo datos del backend:

```typescript
// src/services/api.ts
import { apiService } from './services/api'

// En el componente
onMounted(async () => {
   const data = await apiService.listItems()
  // Usar los datos
})
```

### Variables de entorno:

```bash
# Durante desarrollo
VITE_API_URL=http://localhost:8000

# En producción (Render)
VITE_API_URL=https://tu-api.onrender.com
```

<!-- --- -->

## 4. Estructura del Backend

### Endpoints disponibles:

- **GET /** - Estado del backend
- **GET /api/data** - Estado general y datos de ejemplo
- **GET /api/items** - Listar tareas
- **POST /api/items** - Crear tarea
- **PUT /api/items/{id}** - Actualizar tarea
- **DELETE /api/items/{id}** - Eliminar tarea
- **GET /docs** - Documentación interactiva (Swagger)

### CORS configurado:

El backend permite peticiones desde cualquier origen (configurable en producción):

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # En producción, restringir a dominios específicos
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

<!-- --- -->

## 5. Guía de Despliegue (CI/CD)

El objetivo es que cada vez que hagas un `git push` a la rama `main`, la aplicación se actualice automáticamente en internet.

### A. Despliegue del Backend (Render)

**Configuración para el despliegue automático con GitHub**

1. Ve a [Render.com](https://render.com) e inicia sesión, o crea una cuenta con GitHub.
2. Haz clic en **"New +"** > **"Web Service"**.
3. Conecta tu repositorio de GitHub:
   - Selecciona tu repositorio (`vercel-render`).
   - Haz clic en **"Connect"**.
4. Configura el servicio:
   - **Name:** `vercel-render-backend`
   - **Root Directory:** `./backend` (importante)
   - **Runtime:** `Docker`
   - **Region:** Elige la más cercana a ti
   - **Plan:** Free (o el que prefieras)
5. Haz clic en **"Create Web Service"** y espera a que se despliegue.
6. Una vez desplegado, tu backend estará disponible en una URL como: `https://vercel-render-backend-xxx.onrender.com`

**Configurar Deploy Hook para despliegue automático:**

7. En el dashboard de Render, ve a tu servicio > **Settings**.
8. Busca la sección **"Deploy Hook"** y copia la URL completa.
9. Ve a tu repositorio en GitHub > **Settings** > **Secrets and variables** > **Actions**.
10. Crea un nuevo Secret:
    - **Name:** `RENDER_DEPLOY_HOOK`
    - **Value:** Pega la URL de Render copiada en el paso 8
11. Haz clic en **"Add secret"**.

**¿Qué sucede ahora?**
- Cada vez que hagas `git push` a la rama `main`, GitHub dispara automáticamente el Deploy Hook de Render.
- Render recibe la notificación y comienza un nuevo despliegue.
- Tu backend se actualiza en internet en 1-2 minutos.

**Verifica que funciona:**
- En GitHub, ve a **Actions** después de hacer push.
- Verás los workflows ejecutándose y llamando al Deploy Hook de Render.
- En Render Dashboard, verás nuevos despliegues iniciándose automáticamente.

**Base de Datos MySQL en Producción (Railway):**

- Crea una base de datos MySQL en Railway.
- Copia la cadena de conexión que te proporciona Railway. La puedes encontrar en la sección de variables de entorno de tu proyecto, en la variable `MYSQL_PUBLIC_URL`. 
- En Render > tu servicio backend > **Environment Variables**, agrega:
   - **Name:** `DATABASE_URL`
   - **Value:** `mysql+pymysql://user:password@host:port/database`

Con `DATABASE_URL` definido, el backend usará esa base de datos en producción. En local, seguirá usando las variables `DB_*` de `compose.yaml`.

> Nota: si Railway te entrega la URL con `mysql://`, el backend la normaliza automáticamente a `mysql+pymysql://`.

<!-- --- -->

### B. Despliegue del Frontend (Vercel)

**Despliegue automático mediante GitHub Actions:**

1. Ve a [Vercel.com](https://vercel.com) e inicia sesión (o crea una cuenta con GitHub).
2. Haz clic en **"Add New"** > **"Project"**.
3. Conecta tu repositorio de GitHub:
   - Selecciona tu repositorio (`vercel-render`).
   - Haz clic en **"Import"**.
4. Configura el proyecto:
   - **Project Name:** `vercel-render-frontend` (o el que prefieras)
   - **Root Directory:** `./frontend` (importante, señala la carpeta del frontend)
   - **Framework Preset:** `Vite`
   - **Build Command:** `npm run build` (Vercel lo detecta automáticamente)
   - **Output Directory:** `dist`
5. Configura las variables de entorno:
   - En la sección **"Environment Variables"**, agrega:
     - **Name:** `VITE_API_URL`
     - **Value:** `https://tu-backend.onrender.com` (reemplaza con tu URL de Render)
6. Haz clic en **"Deploy"** y espera a que termine.
7. Una vez desplegado, tu frontend estará disponible en una URL como: `https://vercel-render-frontend-xxx.vercel.app`

**Obtener el token de Vercel para GitHub Actions CI/CD:**

8. Ve a [Vercel Settings > Tokens](https://vercel.com/account/tokens).
9. Haz clic en **"Create"**.
10. Asigna un nombre descriptivo (ej: `GitHub CI/CD`).
11. Selecciona el scope: **Full Account**
12. Haz clic en **"Create Token"** y copia el token completo.

**Configurar el token en GitHub Secrets:**

13. Ve a tu repositorio en GitHub > **Settings** > **Secrets and variables** > **Actions**.
14. Crea un nuevo Secret:
    - **Name:** `VERCEL_TOKEN`
    - **Value:** Pega el token de Vercel copiado
15. Haz clic en **"Add secret"**.

**¿Qué sucede ahora?**
- El workflow `deploy-frontend.yaml` ya está configurado en `.github/workflows/`
- Cada vez que hagas `git push` a la rama `main` (cambios en `frontend/**`), GitHub Actions ejecutará el despliegue
- Vercel recibe los comandos y despliega tu aplicación automáticamente
- Los cambios se verán en la web en 2-3 minutos

**Verifica que funciona:**
- En GitHub, ve a **Actions** después de hacer push
- Verás el workflow `Deploy Frontend to Vercel` ejecutándose
- En Vercel Dashboard, verás nuevos despliegues iniciándose automáticamente

<!-- --- -->

## 6. Conceptos clave

### Docker Multi-stage

En los `Dockerfile`, utilizamos dos o más fases:
- **Builder:** Instala dependencias y compila/construye la aplicación
- **Runner/Development:** Solo copia lo necesario para ejecutar

Esto reduce el tamaño de las imágenes, mejora la seguridad y acelera el despliegue.

### CORS (Cross-Origin Resource Sharing)

El backend está configurado para aceptar peticiones del frontend. Sin esto, el navegador bloquearía la conexión por seguridad.

```python
app.add_middleware(CORSMiddleware, allow_origins=["*"])
```

### Variables de entorno

- **Frontend:** `VITE_API_URL` indica dónde está el backend
- **Backend:** `PORT` indica en qué puerto escuchar
- **Base de datos (local):** `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- **Base de datos (producción):** `DATABASE_URL`

Nunca incluyas estas en el código, usa siempre archivos `.env` (no versionados).

### Secrets de GitHub

Nunca subas contraseñas, tokens o claves al repositorio. Usa siempre:
- GitHub Secrets (Settings > Secrets and variables)
- Variables de entorno en plataformas de despliegue (Vercel, Render)

<!-- --- -->

## 7. Tecnologías utilizadas

| Componente | Tecnología | Versión |
|-----------|-----------|---------|
| **Frontend** | Vue 3 | ^3.4.21 |
| **Build Tool** | Vite | ^5.0.11 |
| **CSS** | Tailwind CSS | ^3.4.1 |
| **Backend** | FastAPI | Última |
| **Python** | Python | 3.11+ |
| **Servidor Docker** | Node.js | 20-slim |
| **Base de Datos** | MySQL | 8.x |

<!-- --- -->

## 8. Comandos útiles durante el desarrollo

```bash
# Desarrollo local
docker compose up --build

# Detener servicios
docker compose down

# Reconstruir solo frontend
docker compose up --build frontend

# Ver logs de un servicio específico
docker compose logs -f frontend

# Ejecutar comando en un contenedor
docker compose exec frontend npm install
```

<!-- --- -->

## 9. Próximos pasos

- [ ] Agregar autenticación con JWT
- [ ] Escribir tests (pytest para backend, Vitest para frontend)
- [ ] Configurar monitoring y logging
- [ ] Documentar APIs con OpenAPI/Swagger

<!-- --- -->

## 10. Licencia

Este proyecto es de código abierto y está disponible bajo la licencia MIT.

# cambios hecho por Rafael 

## Desarrollo local con docker 
En mi caso lo he realizado con AWS en local me daba fallos 

### levantamiento servicios docker 
![alt text](<fotos/desarrollo local con docker/1.levantamiento sevicios docker .png>)

### comprobacion de que el backend funciona y el frontend funciona 
![alt text](<fotos/desarrollo local con docker/2.backend funcionando .png>)
![alt text](<fotos/desarrollo local con docker/3.frontend funcionando .png>)

## despliegue del backend

### despliegue de la base de datos con railway 
![alt text](<fotos/despliegue backend/1.despligue base de datos railway.png>)

### creacion servicio web con render
![alt text](<fotos/despliegue backend/2.creacion servicio web .png>)

### unir a git hub para automatizacion de despligue al realizar un push 
![alt text](<fotos/despliegue backend/4.todo funciona.png>)
![alt text](<fotos/despliegue backend/5.todo funciona 2.png>)

## despliegue del fronted

### creacion proyecto vercel 
![alt text](<fotos/despliegue fronted/1.nuevo proyecto vercel config.png>)

### comprobacion de que funciona 
![alt text](<fotos/despliegue fronted/2.front funcionando.png>)

### creacion de tokens en vercel 
![alt text](<fotos/despliegue fronted/3.TOKENS vercel.png>)

### creacion de los actions necesarios para la automatizacion 
![alt text](<fotos/despliegue fronted/4.todos los actions.png>)

### comprobacion de que funciona al hacer un push 
![alt text](<fotos/despliegue fronted/5. todo funciona 1.png>)

### comprobacion de que funciona render al realizar un cambio
![alt text](<fotos/despliegue fronted/6.todo funciona 2.png>)