# ESCALO — Mapa de direcciones para scan de documentación e infraestructura

> Generado: 2026-07-12. Pegá este documento completo al inicio de la conversación con la IA de análisis, junto con la instrucción de la sección 5.

## 1. Repositorio de código y documentación (GitHub — público)

**URL:** https://github.com/Jair-Infante/ESCALO · rama `main`

Archivos clave a leer:

| Archivo | Qué contiene |
|---|---|
| `Whatsapp agent con n8n y mcp/CLAUDE.md` | Reglas canónicas del proyecto activo (stack, reglas de negocio inamovibles, método de trabajo, manejo de errores) |
| `Whatsapp agent con n8n y mcp/PLAN_ACCION_WHATSAPP.md` | Fuente de verdad del avance: estado actual, fases F0–F6, registro cronológico completo de decisiones |
| `Whatsapp agent con n8n y mcp/docs/RUNBOOK.md` | Operación 24/7: chequeo de salud, reinicio, re-QR, logs, backups |
| `Whatsapp agent con n8n y mcp/docs/CREDENCIALES.md` | Inventario de credenciales y dónde viven (sin valores) |
| `Whatsapp agent con n8n y mcp/docs/COMPORTAMIENTO_IA.md` | Reglas de tono y guiones del agente IA |
| `Whatsapp agent con n8n y mcp/docs/ESCALO_SEGURIDAD_FILTROS.md` | Diseño de los 4 filtros de seguridad (origen, whitelist, pausa humana, debounce/ráfaga) |
| `Whatsapp agent con n8n y mcp/CLAUDE MCP N8N/n8n-workflows/ESCALO_MVP_V2.md` | Spec de negocio V2: esquemas, reglas, casos de prueba obligatorios |
| `Whatsapp agent con n8n y mcp/n8n/exports/*.json` (8 archivos) | Snapshots versionados del workflow n8n, del 2026-07-09 al 2026-07-11 |
| `ESCALO (Whatsapp agent con claude code)/CLAUDE.md` | ⚠️ Carpeta **histórica/deprecada** — el propio archivo dice que el proyecto activo es el de arriba. Útil solo para ver cómo evolucionó el approach. |
| `Web (escalo.com.ar)/index.html` + `docs/` | Landing page y notas de la web |
| `Kit de marca (pendiente)/README.md` | Placeholder, sin desarrollar aún |

## 2. Documentación estratégica (Google Drive — ESCALO_HQ)

**Carpeta raíz:** https://drive.google.com/drive/folders/16ztar9Pbn43rh4RhJYbYi7JfPDYPWu43

| Subcarpeta | Incluir en el scan | Contenido |
|---|---|---|
| `00_INICIO` | Sí | README de orientación, estado "a construir y a definir" |
| `01_Estrategia y Contexto` | Sí | Modelo de negocio y pricing, contexto completo del proyecto, roadmap |
| `02_Producto y Técnico` | Sí | System prompt del agente, prompts MVP V1/V2, runbook de infra, seguridad, workflows n8n IDs |
| `03_Clientes y Pilotos` | **No — excluida a propósito** | Datos personales de clientes/pilotos reales |
| `04_Operaciones` | Sí | Plan de construcción, mapa de accesos (sin valores), checklist de despliegue por cliente nuevo |
| `05_CRM y Datos` | **No — excluida a propósito** | Datos de CRM, probablemente con información personal |
| `06_Web y Marca` | Sí | Landing, kit de marca |

## 3. Infraestructura viva (para verificar estado real, no solo lo que dicen los docs)

| Componente | Dirección |
|---|---|
| VPS | Hostinger KVM2 · `srv1815734.hstgr.cloud` · IP `89.116.225.137` · Ubuntu 24.04 |
| n8n (VPS) | `https://n8n-o324.srv1815734.hstgr.cloud` — workflow activo `ESCALO-MVP-V2`, ID `6IB9N46IpcmpnXmX`, 45 nodos |
| Evolution API (WhatsApp) | `https://evo.escalo.com.ar` · v2.3.7 · instancia `escalo` (Baileys) |
| Webhook | Evolution `MESSAGES_UPSERT` → `https://n8n-o324.srv1815734.hstgr.cloud/webhook/escalo-wa` |
| Dominio | `escalo.com.ar` · DNS en Cloudflare |
| Google Sheets (base de negocio V2) | ID `133rSjQRgaBtv7LZMEiPkDUYpS-0tU89Fsv5Qw8HDGLI` — pestañas `ESCALO_STOCK`, `ESCALO_LEADS`, `ESCALO_HISTORIAL`, `ESCALO_NEGOCIO`, `ESCALO_WHITELIST` |
| Workflow V1 | ID `5CzydNM6GGxtTiFB` — **marcado NO TOCAR** en la documentación, verificar que siga así |

Estos IDs están documentados como "sin secretos" en el propio proyecto — son identificadores, no credenciales.

## 4. Qué NO compartir con la IA de análisis

- `Whatsapp agent con n8n y mcp/.mcp.json` — tiene una API key de n8n en texto plano (ya excluida del repo por `.gitignore`, no la subas a ningún lado).
- Contenido literal de `docs/CREDENCIALES.md` más allá del inventario — es metadata, no pidas que se completen los `[COMPLETAR]` con valores reales en el chat.
- Carpetas Drive `03_Clientes y Pilotos` y `05_CRM y Datos`.

## 5. Protocolo de scan sugerido — instrucción para pegarle a la IA

> "Tenés acceso a este mapa de direcciones de ESCALO, un SaaS de agente IA para WhatsApp (vertical automotor). Quiero que:
> 1. Compares el estado declarado en `PLAN_ACCION_WHATSAPP.md` contra lo que muestran los exports de `n8n/exports/` y `docs/RUNBOOK.md` — señalá cualquier inconsistencia de fechas o de estado.
> 2. Identifiques deuda técnica y pendientes abiertos (F5.3, F5.4, F6, el Bloque C de alertas Telegram+Email que depende de que yo cree el bot).
> 3. Evalúes riesgos de seguridad y de negocio: uso de Baileys no oficial, whitelist restrictiva bloqueando leads reales, filtros fail-closed, manejo de errores en dos capas.
> 4. Señales duplicación entre la carpeta activa (`Whatsapp agent con n8n y mcp`) y la histórica (`ESCALO (Whatsapp agent con claude code)`) — confirmá que la segunda ya no aporta nada vigente o decime si hay algo rescatable.
> 5. Contrastes el roadmap/estrategia de `01_Estrategia y Contexto` en Drive contra la ejecución real registrada en el repo — gaps entre lo planeado y lo hecho.
> 6. Marques huecos de documentación (campos `[COMPLETAR]` sin llenar en `CREDENCIALES.md` y `RUNBOOK.md`).
> No necesitás ni debés pedir valores de credenciales — todo lo relevante para este análisis es estructural, no secretos."
