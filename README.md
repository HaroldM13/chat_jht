# JHT Chat

Aplicación de chat en tiempo real. Backend con FastAPI + MongoDB + Redis + RabbitMQ. Frontend con React + TypeScript.

## Prerrequisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y corriendo

Eso es todo.

## Levantar la aplicación

```bash
# 1. Clonar el repo (incluye backend y frontend automáticamente)
git clone --recurse-submodules https://github.com/TU_USUARIO/jht-chat
cd jht-chat

# 2. Configurar variables de entorno (opcional para uso local)
cp .env.example .env

# 3. Levantar todo
docker compose up -d
```

## Abrir en el navegador

| Servicio | URL |
|---|---|
| **App** | http://localhost:5173 |
| RabbitMQ (admin) | http://localhost:15672 — usuario: `guest` / contraseña: `guest` |
| Redis Commander | http://localhost:8081 |
| Dozzle (logs) | http://localhost:9999 |

## Comandos útiles

```bash
# Ver estado de los contenedores
docker compose ps

# Ver logs en tiempo real
docker compose logs -f backend
docker compose logs -f frontend

# Detener todo
docker compose down

# Detener y borrar la base de datos
docker compose down -v
```

## Estructura

```
jht-chat/
├── docker-compose.yml
├── .env.example
├── backend_chat/     ← API FastAPI
└── frontend_chat/    ← App React
```
