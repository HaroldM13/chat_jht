# JHT Chat

Aplicación de chat en tiempo real. Backend con FastAPI + MongoDB + Redis + RabbitMQ. Frontend con React + TypeScript.

Todo corre dentro de Docker: no hace falta instalar Python, Node, MongoDB ni nada más.

---

## Requisito previo

Solo necesitás tener instalado **Docker Desktop** (incluye Docker Compose):

- **Windows / Mac**: https://www.docker.com/products/docker-desktop/
- **Linux (Ubuntu/Debian)**:
  ```bash
  sudo apt update
  sudo apt install -y docker.io docker-compose-plugin
  sudo systemctl start docker
  sudo systemctl enable docker
  sudo usermod -aG docker $USER
  # Cerrar sesión y volver a abrir para que el grupo tenga efecto
  ```

Verificar que Docker está corriendo:
```bash
docker --version
docker compose version
```

---

## Paso 1 — Clonar el repositorio

Las imágenes del backend y frontend ya están publicadas en DockerHub y se descargan automáticamente. Solo necesitás el `docker-compose.yml` y el `.env`, así que el clone es simple:

**Linux / Mac / WSL:**
```bash
git clone https://github.com/HaroldM13/chat_jht.git
cd chat_jht
```

**Windows (PowerShell o CMD):**
```powershell
git clone https://github.com/HaroldM13/chat_jht.git
cd chat_jht
```

---

## Paso 2 — Configurar variables de entorno

Hay un archivo `.env.example` con los valores por defecto listos para uso local. Copiarlo a `.env`:

**Linux / Mac / WSL:**
```bash
cp .env.example .env
```

**Windows (PowerShell o CMD):**
```powershell
copy .env.example .env
```

> Para uso local no hace falta editar nada. Los valores del `.env.example` ya apuntan a `localhost` y funcionan sin cambios.

---

## Paso 3 — Levantar la aplicación

```bash
docker compose up -d
```

Este comando es igual en Linux, Mac y Windows. La primera vez puede tardar unos minutos mientras descarga las imágenes de DockerHub (`docker pull` ocurre automáticamente).

Verificar que todo arrancó bien:
```bash
docker compose ps
```

Todos los servicios deben aparecer con estado `running` o `Up`.

---

## Paso 4 — Abrir en el navegador

| Servicio | URL |
|---|---|
| **App (chat)** | http://localhost:5173 |
| RabbitMQ (admin) | http://localhost:15672 — usuario: `guest` / contraseña: `guest` |
| Redis Commander | http://localhost:8081 |
| Dozzle (logs) | http://localhost:9999 |

> Si el frontend muestra errores al principio, esperá 10-15 segundos y recargá la página. El backend necesita unos segundos para conectarse a todos los servicios.

---

## Comandos útiles

```bash
# Ver el estado de todos los contenedores
docker compose ps

# Ver logs en tiempo real
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f backend
docker compose logs -f frontend

# Detener todo (los datos se conservan)
docker compose down

# Detener y borrar todos los datos
docker compose down -v

# Reiniciar un servicio
docker compose restart backend
```

---

## Estructura del proyecto

```
chat_jht/
├── .github/
│   └── workflows/
│       └── docker.yml    ← pipeline CI/CD automático
├── docker-compose.yml    ← orquesta todos los servicios
├── .env.example          ← plantilla de configuración
├── backend_chat/         ← API FastAPI (submódulo)
└── frontend_chat/        ← App React (submódulo)
```

---

## Problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `docker: command not found` | Docker no está instalado | Instalar Docker Desktop |
| `permission denied` al correr docker (Linux) | Usuario no está en el grupo docker | `sudo usermod -aG docker $USER` y cerrar/abrir sesión |
| El frontend carga pero no conecta al backend | El backend todavía está iniciando | Esperar 15 segundos y recargar |
| Puerto 5173 u 8000 ya en uso | Otro proceso ocupa ese puerto | `docker compose down` y volver a levantar |
| `docker compose up` falla en Windows | Docker Desktop no está corriendo | Abrir Docker Desktop y esperar que inicie |
| `pull access denied` al levantar | La imagen no existe en DockerHub | Verificar que el CI/CD corrió exitosamente en GitHub Actions |

---

## CI/CD — Preguntas del proyecto

### ¿Qué automatizaron?

Automatizamos dos cosas:

**1. El despliegue local** — con `docker compose up -d` se levantan automáticamente 7 servicios (MongoDB, Redis, RabbitMQ, backend FastAPI, frontend React con nginx, Dozzle y Redis Commander) en el orden correcto. Los healthchecks garantizan que el backend no arranque hasta que la base de datos, Redis y RabbitMQ estén listos.

**2. La construcción y publicación de imágenes** — con GitHub Actions, cada vez que se hace `git push` a la rama `main`, se construyen automáticamente las imágenes Docker del backend y el frontend y se publican en DockerHub. Nadie tiene que hacer `docker build` ni `docker push` manualmente.

---

### ¿Cómo funciona el flujo?

```
Desarrollador hace git push a main
        ↓
GitHub Actions detecta el push y lanza el pipeline
        ↓
1. Descarga el código con los submódulos (backend y frontend)
2. Se autentica en DockerHub
3. Construye la imagen del backend desde backend_chat/Dockerfile
4. Sube la imagen → dockerhub: usuario/chat-backend:latest
5. Construye la imagen del frontend desde frontend_chat/Dockerfile
6. Sube la imagen → dockerhub: usuario/chat-frontend:latest
        ↓
Cualquier persona puede levantar el sistema completo con:
  git clone → cp .env.example .env → docker compose up -d
```

El `docker-compose.yml` del orquestador define la red interna entre servicios, los volúmenes para persistencia de datos y las dependencias entre contenedores.

---

### ¿Qué ventajas tiene CI/CD?

| Ventaja | Sin CI/CD | Con CI/CD |
|---|---|---|
| **Consistencia** | "En mi máquina funciona" | La misma imagen corre en todas las máquinas |
| **Velocidad** | Build y push manual en cada cambio | Automático en cada `git push` |
| **Errores humanos** | Posible olvidar pasos o subir versión incorrecta | El pipeline siempre sigue los mismos pasos |
| **Despliegue** | Requiere acceso al servidor y comandos manuales | Cualquier máquina levanta el sistema con 3 comandos |
| **Trazabilidad** | No se sabe qué versión está corriendo | Cada imagen tiene su tag y se puede rastrear al commit |

En resumen: el sistema deja de depender de una máquina específica o de que alguien sepa los pasos exactos. Cualquier persona con Docker puede clonar y levantar el sistema completo en minutos.
