# RUNBOOK — Operación ESCALO 24/7

> Esqueleto: cada sección se completa al ejecutar la fase indicada del plan (`[COMPLETAR Fx]`).
> Mantenerlo corto y accionable: es el documento que se lee con el sistema caído.

## Chequeo rápido de salud

- [ ] n8n VPS responde: `https://n8n-o324.srv1815734.hstgr.cloud` → pestaña Executions sin errores nuevos
- [ ] Evolution responde: `curl https://evo.escalo.com.ar` → JSON de bienvenida v2.3.7. Estado de instancia: `curl https://evo.escalo.com.ar/instance/connectionState/escalo -H "apikey: $EVO_API_KEY"` → `state: open`
- [ ] En el VPS: `docker ps` → `n8n-o324-n8n-1`, `traefik-traefik-1`, `evolution-evolution-api-1`, `evolution-evo-postgres-1`, `evolution-evo-redis-1` todos `Up`
- [ ] Prueba real: enviar "hola" al número de prueba → responde en menos de 60 segundos

## Reinicio seguro

- Evolution: `cd /root/evolution && docker compose restart` (los 3 contenedores tienen `restart: always`).
- n8n: reiniciar desde panel Hostinger → Docker.
- La sesión de WhatsApp debe sobrevivir reinicios (persistida en volumen `evolution_instances` + Postgres). Verificado en F6.1: [pendiente]

## WhatsApp desconectado / re-QR

- Síntoma: instancia con estado ≠ `open`, los mensajes no llegan a n8n.
- QR fresco por API: `curl https://evo.escalo.com.ar/instance/connect/escalo -H "apikey: $EVO_API_KEY"` (devuelve `base64` con el PNG; rota cada ~40s). O manager `https://evo.escalo.com.ar/manager` (login con la API key, obtenerla por SSH: `grep EVO_API_KEY /root/evolution/.env`).
- Escanear con el número de prueba: WhatsApp → Dispositivos vinculados → Vincular dispositivo.
- Si el QR falla repetidamente: fijar `CONFIG_SESSION_PHONE_VERSION` en el compose a la versión actual de WhatsApp Web y recrear el contenedor.

## Logs

- Evolution: `docker logs evolution-evolution-api-1 --tail 100`
- n8n: pestaña Executions (error por ejecución, con datos de entrada) + `docker logs n8n-o324-n8n-1 --tail 100`

## Webhook Evolution → n8n

- Configurado: evento `MESSAGES_UPSERT` → `https://n8n-o324.srv1815734.hstgr.cloud/webhook/escalo-wa` (workflow `ESCALO-MVP-V2` debe estar **activo**).
- Verificar config: `curl https://evo.escalo.com.ar/webhook/find/escalo -H "apikey: $EVO_API_KEY"`
- Reconfigurar: `POST /webhook/set/escalo` con `{"webhook":{"enabled":true,"url":"...","events":["MESSAGES_UPSERT"]}}`.
- Nota: el webhook de n8n responde al terminar el pipeline (~10-20s, modo `responseNode`) — es normal, Evolution no depende de la respuesta.

## Filtro de grupos (Evolution)

- La instancia `escalo` está configurada con `groupsIgnore: true` (2026-07-13): Evolution **no emite** mensajes de grupo, así que n8n nunca los recibe. Es el corte de raíz para que la IA ni lea grupos.
- Verificar: `curl https://evo.escalo.com.ar/settings/find/escalo -H "apikey: $EVO_API_KEY"` → `"groupsIgnore":true`.
- Revertir (volver a recibir grupos): `POST /settings/set/escalo` con el mismo body pero `"groupsIgnore":false`.
- Defensa en profundidad: aunque un grupo se filtre igual, el nodo `WA → normalizado` lo clasifica `descartar/grupo` y corta antes de la IA.

## Backups

- Workflows: exportar a `n8n/exports/` + commit en cada sesión productiva.
- Datos de negocio: las Sheets SON la base de datos → historial de versiones de Google activo por defecto; no borrar ni renombrar pestañas.
- Postgres de Evolution: `[COMPLETAR F6]` — evaluar dump semanal cuando haya número real.

## Incidentes y soluciones (vivo)

- (agregar acá con fecha a medida que ocurran)
