# CREDENCIALES — Inventario (SIN valores)

> Regla: acá NUNCA se escriben keys, tokens ni contraseñas — solo qué existe, dónde vive y dónde se usa.
> Si una key aparece en un chat, log o commit: se rota de inmediato.

## Cuentas y servicios

| Servicio | Cuenta/dueño | Acceso | Notas |
|---|---|---|---|
| Google Cloud (OAuth client) | [COMPLETAR email] | console.cloud.google.com | Client OAuth de Sheets/Calendar para n8n. Redirect URIs autorizados: `http://localhost:5678/rest/oauth2-credential/callback` (local) — **agregar la del n8n VPS en F3.3**: `https://<n8n-vps>/rest/oauth2-credential/callback` |
| Anthropic | [COMPLETAR email] | console.anthropic.com | Key usada por la credencial n8n "Anthropic account" |
| Hostinger (VPS) | [COMPLETAR email] | hpanel.hostinger.com | KVM2 · `srv1815734.hstgr.cloud` · `89.116.225.137` · SSH root |
| Cloudflare | [COMPLETAR email] | dash.cloudflare.com | Zona `escalo.com.ar` (plan Free) |
| NIC.ar | CUIT/Clave Fiscal de Jair | nic.ar | Dominio `escalo.com.ar`, delegado a Cloudflare (alexis/rayne) |
| WhatsApp de prueba | Número: [COMPLETAR] | Teléfono de Jair | La sesión Baileys se persiste en el volumen `evolution_instances` + Postgres del VPS |

## Credenciales dentro de n8n local (`localhost:5678`)

| Nombre en n8n | Tipo | ID | Usada por |
|---|---|---|---|
| Google Sheets account MVP v2 | OAuth2 | `sXgC2e9hZ6fNiJ2w` | Los 4 nodos Sheets de V2 |
| Google Calendar account | OAuth2 | `UkLaBd8XfSYCMPc8` | Ningún nodo todavía (post-MVP) |
| Anthropic account | API key | `lw5EBdHsTv1YKwwx` | Anthropic Chat Model |

Al migrar (F3.3) se recrean las tres en el n8n del VPS — los valores **no viajan** con el export del workflow.

## Credenciales dentro de n8n VPS (`n8n-o324.srv1815734.hstgr.cloud`)

| Nombre en n8n | Tipo | ID | Usada por |
|---|---|---|---|
| Google Sheets account MVP v2 | OAuth2 | `AP6CbVAiiS4GkpkH` | Los 6 nodos Sheets/HTTP de ESCALO-MVP-V2 (asignada en F3.3) |
| Google Calendar account | OAuth2 | `1JsQspH61R9PDPn2` | Ningún nodo todavía (post-MVP) |
| Anthropic account | API key | `kbeF2OZ0AviWwmPH` | Anthropic Chat Model (asignada en F3.3) |
| Evolution API escalo | Header Auth (header `apikey`) | `7spabdoxFvIF01rq` | Nodo `Enviar WhatsApp` de ESCALO-MVP-V2 (creada 2026-07-10 vía API REST, valor tomado del `.env` del VPS sin pasar por chat) |

Nota: al recrear la credencial OAuth2 de Google, verificar que el redirect URI del n8n VPS esté agregado en el OAuth client de Google Cloud Console (ver fila de arriba).

## API keys de infraestructura

| Key | Dónde vive | Rotación |
|---|---|---|
| n8n API (local) | `.mcp.json` raíz del proyecto (en `.gitignore`, no se commitea) | Muere con el n8n local después de la migración |
| n8n API (VPS) | En `.mcp.json` (confirmada operativa 2026-07-09, F3.5) | Rotar si se expone; actualizar `.mcp.json` |
| Evolution API (`AUTHENTICATION_API_KEY` = `EVO_API_KEY`) | `/root/evolution/.env` en el VPS (creada 2026-07-10, permisos 600) | Rotar = editar `.env` + `docker compose up -d` + actualizar la credencial `Evolution API escalo` en n8n |
| Evolution apikey en n8n VPS (credencial **Header Auth**, header `apikey`) | Credencial `Evolution API escalo` (`7spabdoxFvIF01rq`), asignada al nodo `Enviar WhatsApp` | Misma rotación que la key de Evolution |
| Postgres Evolution (`EVO_DB_PASS`) | `/root/evolution/.env` del VPS (creada 2026-07-10) | Solo accesible dentro de la red Docker del VPS |
