# estados_apis (infra)

Stack de Docker Swarm de **Estados APIS**, el panel de monitoreo de APIs.

- Código, `Dockerfile` y guía completa: https://github.com/IntegracionesFlexxus/estados_apis (`docker/README.md`)
- Acceso: `http://10.250.0.98:3255` (puerto publicado en el worker)
- Base de datos: SQLite en el volumen `estados-apis-data` del worker (1 réplica fijada por `NODE`)

## Antes del primer deploy

**Secrets** (Portainer → *Secrets*), con los valores del `.env` del servidor y sin salto de línea al final:

| Secret | Variable en la app |
|---|---|
| `estados_apis_bot_token` | `BOT_TOKEN` |
| `estados_apis_bot_chat_id` | `BOT_CHAT_ID` |
| `estados_apis_fb_password` | `FBPASSWORD_SAR` |
| `estados_apis_gmail` | `GMAIL` |

**Requisitos del worker:** red overlay `traefik`, plugin de logging `loki`, puerto 3255 libre en el Swarm y acceso a `192.168.121.178:3050` (Firebird del SAR).

## Variables del stack

| Variable | Obligatoria | Valor |
|---|---|---|
| `IMAGE` | Sí | `<usuario>/estados_apis:<tag>` |
| `NODE` | Sí | El mismo que usan los otros stacks del worker, ej. `node.hostname==...` |
| `LOKI_URL` | Sí | La misma que usan los otros stacks del worker |
| `APP_PORT` | No | Puerto publicado (default `3255`) |

## Actualizar

Construir y subir la imagen con un tag nuevo, cambiar `IMAGE` y *Update the stack*. El contenedor viejo se baja antes de levantar el nuevo (corte de unos segundos, por SQLite) y si el nuevo no pasa el healthcheck vuelve atrás solo.

## Verificar

```sh
curl http://10.250.0.98:3255/api/health   # {"ok":true,"db":true}
```

## Backups

Los datos viven en el disco del worker. Respaldar con **Exportar APIs** desde el panel o con `VACUUM INTO` (comando en `docker/README.md` del repo del código).
