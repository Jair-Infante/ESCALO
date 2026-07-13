# PLAN DE ACCIÓN — ESCALO a WhatsApp real (Evolution API)

> **Objetivo:** ESCALO-MVP-V2 respondiendo por WhatsApp real desde el VPS, con los casos de prueba obligatorios pasando.
> **Última actualización:** 2026-07-12 · **PASO ACTUAL: SEC.6 y SEC.7 cerrados (4/4 filtros probados, caso 3 confirmado con tráfico real de producción; workflow ACTIVADO confirmado por API). Próximo: F5.3/F5.4 — decidir con Jair si ampliar `ESCALO_WHITELIST` antes**

## Protocolo de uso (leer siempre)

1. Al iniciar sesión: leer SOLO este encabezado, «Estado actual» y la fase en curso. No releer fases ya completadas.
2. Al completar cada tarea: Claude marca el checkbox, actualiza «PASO ACTUAL» y agrega una línea al Registro.
3. Claude informa el ID del paso al cerrar cada tarea. Jair lo dice al retomar tras `/clear` («estamos en F2.1»).
4. Una fase por sesión aproximadamente. Cada tarea ≤ 1 hora.
5. Todo cambio de infra/credenciales se refleja en `docs/RUNBOOK.md` y `docs/CREDENCIALES.md` **en el momento** (con contexto fresco cuesta casi nada).

## Estado actual

- **Paso actual:** **SEC.6 y SEC.7 cerrados** (2026-07-12): ver Registro. Caso 3 (pausa por intervención humana) confirmado sin que Jair tuviera que hacer una prueba deliberada — se validó solo, con tráfico real: ejecución n8n `261` (21:16:39hs), contacto real `5492944147534`, payload de Evolution con `fromMe:true`, `source:"ios"`, `pushName:"Jair."` → `WA → normalizado` clasificó `ruta:humano/motivo:intervencion_humana` y `Pausar Contacto (humano)` creó la fila en `escalo_pausas` con `hasta_ts - creado_ts` = exactamente 14.400.000 ms (4h) ✅. Con los 4 casos probados (grupo, whitelist, pausa humana, debounce+ráfaga), **SEC.6 cerrado**. **SEC.7** verificado por `n8n_get_workflow`: `active: true` (nunca se desactivó durante las pruebas, no hizo falta reactivar). **Hallazgo de negocio para decidir con Jair:** el mismo tráfico de hoy muestra 5 contactos reales distintos (no el número de prueba) con fila `motivo:humano` en `escalo_pausas` — como `ESCALO_WHITELIST` solo tiene el número de Jair, el bot los descartó a todos en silencio (fail-closed correcto, SEC.2) y Jair los atendió a mano por WhatsApp (uno de los mensajes: "avísame cuando quieras tu alfajor" — real, no prueba). Ningún lead quedó sin respuesta (Jair respondió manual), pero el bot no ayudó en ninguno de esos 5 mientras la whitelist siga restringida a su propio número. Próximo: **F5.3/F5.4** — casos de prueba obligatorios por WhatsApp real; decidir con Jair si ampliar la whitelist antes o en paralelo. Pendiente **Bloque C** (flujo separado de alertas Telegram+Email+`ESCALO_ERRORES`, ver Registro) — requiere que Jair cree el bot de Telegram, defina el mail y la pestaña.
- **Comportamiento IA:** doc canónico en `docs/COMPORTAMIENTO_IA.md` (2026-07-10): tono porteño natural, preguntar más/informar menos, guiones FINANCIACION (DNI + anticipo) y ENTREGA_USADO (peritaje + datos para tasación), TRABAJO_CV/POSVENTA_SERVICE con respuesta fija, OBSERVACIONES IA sin fecha con registro de cambios de interés/temperatura. Regla estricta MEDIOS DE PAGO (2026-07-10, testeada ✅): pregunta general → no listar, decir que hay varias formas y preguntar cómo pensaba manejarse; pregunta puntual → confirmar solo ese dato, prohibido mencionar otros medios.
- **Workflow VPS:** `ESCALO-MVP-V2` ID `6IB9N46IpcmpnXmX` (45 nodos, **ACTIVADO** — reactivado 2026-07-12 para pruebas SEC.6, ver Registro). Camino WA: `Webhook WhatsApp` (POST `/webhook/escalo-wa`, `responseMode: responseNode`) → `WA → normalizado` → empalma en `1. Normalizar`. Salida: `Ensamblar JSON Final` → IF `¿Enviar por WhatsApp?` → true: `Enviar WhatsApp` (credencial `Evolution API escalo`, id `7spabdoxFvIF01rq`) → `Responder Webhook` / false: `Responder Webhook`. **Cambio F4.5:** ambos webhooks en modo `responseNode` y la rama true también cierra en `Responder Webhook` — n8n rechaza "respond immediately" si un nodo Respond es alcanzable ("Unused Respond to Webhook node"). Implicancia: Evolution espera la respuesta del pipeline (~10-20s), aceptable para MVP.
- **Evolution API:** v2.3.7 corriendo en el VPS (`/root/evolution/docker-compose.yml`: evolution-api + postgres 16 + redis 7, `restart: always`). HTTPS OK vía Traefik (`https://evo.escalo.com.ar`, cert Let's Encrypt emitido). Secretos en `/root/evolution/.env` (EVO_API_KEY, EVO_DB_PASS). Instancia `escalo` creada (Baileys), webhook `MESSAGES_UPSERT` → `https://n8n-o324.srv1815734.hstgr.cloud/webhook/escalo-wa` configurado. **Estado: `open` — QR escaneado (F2.2 ✅).**
- **Tests de regresión (2026-07-10):** (a) paridad por `/escalo-v2-test` ✅ (caso #7, misma forma de JSON, datos exactos del stock, 0 errores); (b) `messages.upsert` simulado por `/webhook/escalo-wa` ✅ hasta `Enviar WhatsApp`, que falla con 500 de Evolution porque la instancia no tiene WhatsApp vinculado — **esperado**, se resuelve al escanear el QR.

---

## Datos clave (sin secretos)

| Qué | Valor |
|---|---|
| VPS | Hostinger KVM2 · `srv1815734.hstgr.cloud` · IP `89.116.225.137` · Ubuntu 24.04 |
| Contenedores existentes | `n8n-o324` (n8n) + `traefik` (reverse proxy) — panel Hostinger → Docker |
| Dominio | `escalo.com.ar` · DNS en Cloudflare (NS alexis/rayne) · A `@` → IP, DNS only |
| Subdominio Evolution | `evo.escalo.com.ar` (crear en F0.2) |
| n8n VPS | queda en su URL `*.hstgr.cloud` para el MVP (menos fricción con Google OAuth) |
| Workflow V2 local | ID `Rb95ZhoeHWeOAWzp` en `localhost:5678` |
| Workflow V1 | `5CzydNM6GGxtTiFB` — **NO TOCAR** |
| Spreadsheet V2 | `133rSjQRgaBtv7LZMEiPkDUYpS-0tU89Fsv5Qw8HDGLI` · HISTORIAL: header en **fila 2** |
| Credenciales (nombres) | "Google Sheets account MVP v2" · "Google Calendar account" · "Anthropic account" |
| Modelo IA | `claude-haiku-4-5` (fijo) |
| Casos de prueba | sección "Casos de prueba obligatorios" de `CLAUDE MCP N8N/n8n-workflows/ESCALO_MVP_V2.md` |

---

## F0 — Preparación (≈30 min)

- [x] **F0.1** Propagación DNS confirmada 2026-07-10: `escalo.com.ar` y `evo.escalo.com.ar` → `89.116.225.137` en 1.1.1.1 y 8.8.8.8. (El registro `evo` hubo que recrearlo: no existía en la zona pese a F0.2 — Jair lo recargó.)
- [x] **F0.2** [Jair, 2 min] Crear en Cloudflare registro **A** `evo` → `89.116.225.137`, **DNS only (nube gris)**
- [x] **F0.3** Acceso al VPS definido: SSH desde esta máquina. Clave `~/.ssh/escalo_vps` (ed25519, comentario `escalo-vps-jair`), usuario `root`, puerto 22. Conexión probada OK: `ssh -i ~/.ssh/escalo_vps root@srv1815734.hstgr.cloud`.
- [x] **F0.4** Setup existente inspeccionado vía SSH:
  - Contenedores: `n8n-o324-n8n-1` (n8n) + `traefik-traefik-1`, proyecto compose `/docker/n8n-o324/docker-compose.yml`
  - **Traefik corre en `network_mode: host`** (no hay red Docker dedicada que unir) · descubre contenedores vía socket Docker (`/var/run/docker.sock`, solo lectura) · `exposedbydefault=false` (hay que poner `traefik.enable=true` explícito)
  - **certresolver = `letsencrypt`**, desafío **HTTP-01 por puerto 80** (`entrypoints.web` → redirige a `websecure`) → **el certificado de `evo.escalo.com.ar` no se puede emitir hasta que el DNS público resuelva** (confirma que F1.3 espera propagación)
  - URL pública n8n VPS = `https://n8n-o324.srv1815734.hstgr.cloud`
  - Implicancia para F1.2: en el docker-compose de Evolution, **no hace falta `networks: [..., <RED-TRAEFIK>]` ni `external: true`** — alcanza con las labels de Traefik (`traefik.enable=true`, router, certresolver `letsencrypt`); el contenedor puede quedar en su red `default` propia, Traefik lo alcanza vía el socket Docker.

## F1 — Instalar Evolution API en el VPS (≈1h)

- [x] **F1.1** Hecho 2026-07-10: `/root/evolution/.env` con `EVO_API_KEY` (hex 32) y `EVO_DB_PASS` (hex 16), permisos 600, generados en el VPS sin pasar por el chat.
- [x] **F1.2** Hecho 2026-07-10: `/root/evolution/docker-compose.yml` con imagen **`evoapicloud/evolution-api:v2.3.7`** (fijada; 2.4.0 aún RC). Según F0.4 no hizo falta red de Traefik: solo labels (`certresolver=letsencrypt`, puerto 8080). Plantilla original de referencia:

  ```yaml
  services:
    evolution-api:
      image: evoapicloud/evolution-api:latest   # verificar última v2 estable
      restart: always
      networks: [default, <RED-TRAEFIK>]
      environment:
        - SERVER_URL=https://evo.escalo.com.ar
        - AUTHENTICATION_API_KEY=${EVO_API_KEY}
        - LANGUAGE=es
        - DATABASE_ENABLED=true
        - DATABASE_PROVIDER=postgresql
        - DATABASE_CONNECTION_URI=postgresql://evolution:${EVO_DB_PASS}@evo-postgres:5432/evolution?schema=public
        - DATABASE_SAVE_DATA_INSTANCE=true
        - CACHE_REDIS_ENABLED=true
        - CACHE_REDIS_URI=redis://evo-redis:6379/6
        - CACHE_REDIS_PREFIX_KEY=evolution
        - CACHE_LOCAL_ENABLED=false
      labels:
        - traefik.enable=true
        - traefik.http.routers.evolution.rule=Host(`evo.escalo.com.ar`)
        - traefik.http.routers.evolution.entrypoints=websecure
        - traefik.http.routers.evolution.tls.certresolver=<CERTRESOLVER>
        - traefik.http.services.evolution.loadbalancer.server.port=8080
      volumes:
        - evolution_instances:/evolution/instances
      depends_on: [evo-postgres, evo-redis]
    evo-postgres:
      image: postgres:16-alpine
      restart: always
      environment:
        - POSTGRES_USER=evolution
        - POSTGRES_PASSWORD=${EVO_DB_PASS}
        - POSTGRES_DB=evolution
      volumes:
        - evo_pgdata:/var/lib/postgresql/data
    evo-redis:
      image: redis:7-alpine
      restart: always
      volumes:
        - evo_redisdata:/data
  volumes:
    evolution_instances:
    evo_pgdata:
    evo_redisdata:
  networks:
    <RED-TRAEFIK>:
      external: true
  ```
- [x] **F1.3** Hecho 2026-07-10: stack arriba (`evolution-api` + `evo-postgres` + `evo-redis`), logs limpios (Redis ready, Prisma ON, HTTP 8080), `https://evo.escalo.com.ar` responde el JSON de bienvenida v2.3.7 con certificado válido. Auth verificada: sin `apikey` → 401.

## F2 — Conectar WhatsApp de prueba (≈30 min)

- [x] **F2.1** Hecho 2026-07-10: instancia `escalo` creada vía `POST /instance/create` (`WHATSAPP-BAILEYS`, `qrcode: true`), estado `connecting`.
- [x] **F2.2** Hecho 2026-07-10: QR escaneado con el número de PRUEBA de Jair. `GET /instance/connectionState/escalo` → `{"instance":{"instanceName":"escalo","state":"open"}}`.
- [x] **F2.3** Hecho 2026-07-10: en vez de smoke test sintético, Jair mandó un mensaje real ("hola! Quiero saber sobre un Corolla 2022 SEG HEV") a la instancia `escalo`. Ejecución `id 6` en n8n: 17/17 nodos OK, IA clasificó (STOCK, TIBIO, escaló por trim no confirmado), guardó lead + historial, `Enviar WhatsApp` devolvió `status: PENDING` y Jair confirmó que **recibió la respuesta** en su teléfono. Pipeline completo validado en producción.

## F3 — Migrar V2 al n8n del VPS (≈1h)

- [x] **F3.1** Exportado vía MCP (`n8n_get_workflow` modo full, más preciso que UI → Download) → `n8n/exports/ESCALO-MVP-V2_2026-07-09.json`.
- [x] **F3.2** Importado por Jair en n8n VPS (UI → Import from File). Visible en Workflows, no activado. Falta F3.3 (credenciales).
- [x] **F3.3** Credenciales recreadas y asignadas en el VPS: "Google Sheets account" (`AP6CbVAiiS4GkpkH`, en los 6 nodos Sheets/HTTP) y "Anthropic account" (`kbeF2OZ0AviWwmPH`, en Anthropic Chat Model). Verificado por MCP (`n8n_get_workflow` filtered).
- [x] **F3.4** Test de paridad OK (2026-07-09, vía `n8n_test_workflow`, caso de prueba #7 — "quiero ir a ver el Corolla el sábado a las 15hs", contacto `5491112345678`):
  - Respuesta 200, misma forma de JSON que en local (`ok/workflow/lead/analisis/accion/respuesta/errores`).
  - IA no inventó datos: color "Blanco", año 2022, "Full equipo, 45.000 km" — coinciden exacto con `ESCALO_STOCK` fila 6 (Corolla). No mencionó precio (no fue preguntado).
  - `Guardar Lead` escribió fila en `ESCALO_LEADS` (contacto `5491112345678`, estado CALIENTE, intención AGENDAR_VISITA).
  - `Guardar Historial` agregó fila `ESCALO_HISTORIAL!A8:R8`.
  - Fila de prueba pendiente de limpieza en F5.2 (ya estaba previsto).
  - Workflow se activó para poder testear vía webhook y se desactivó de nuevo al terminar (decisión de Jair).
- [x] **F3.5** MCP local reapuntado y confirmado (2026-07-09): `.mcp.json` → `https://n8n-o324.srv1815734.hstgr.cloud`, API key válida (`n8n_health_check` OK, lista workflows). Claude opera el n8n del VPS por MCP.

## F4 — Adaptar workflow a Evolution (≈1h, Claude vía MCP)

- [x] **F4.1** Nodo `Webhook WhatsApp` creado: path `escalo-wa`, POST, `responseMode: onReceived` (200 inmediato).
- [x] **F4.2** Nodo Code `WA → normalizado` creado, mapeo del evento `messages.upsert`:
  - `contacto` = `data.key.remoteJid` antes de `@` (solo dígitos — coherente con "1. Normalizar")
  - `nombre` = `data.pushName` · `canal` = `"whatsapp"` · `timestamp` = `data.messageTimestamp` → ISO
  - `mensaje` = `data.message.conversation` o `data.message.extendedTextMessage.text`
  - **Descartar sin procesar:** `fromMe = true` · grupos (`remoteJid` termina en `@g.us`) · sin texto (audio/imagen → ver Riesgos)
- [x] **F4.3** Conectado a "1. Normalizar"; webhook de simulación `/escalo-v2-test` sigue vivo (rama false del IF de salida → `Responder Webhook`).
- [x] **F4.4** Nodo `Enviar WhatsApp` creado: `POST https://evo.escalo.com.ar/message/sendText/escalo`, auth `httpHeaderAuth` (⚠️ credencial pendiente de crear — necesita la API key de Evolution de F1.1), body `{ number, text }` vía JSON.stringify. Gate: IF `¿Enviar por WhatsApp?` (canal=whatsapp Y respuesta.cliente no vacía); los descartes ya cortan en `WA → normalizado` (return []).
- [x] **F4.5** Hecho 2026-07-10 (OK de Jair): credencial `Evolution API escalo` (`7spabdoxFvIF01rq`, httpHeaderAuth `apikey`, creada vía API REST sin exponer la key) asignada a `Enviar WhatsApp`; workflow **activado**; tests de regresión (a) ✅ y (b) ✅-parcial (falla esperada en `Enviar WhatsApp` por instancia sin vincular). **Fix necesario:** webhook WA pasó de `onReceived` a `responseNode` + conexión `Enviar WhatsApp` → `Responder Webhook` (n8n rechaza "respond immediately" con un Respond node alcanzable). Export: `n8n/exports/ESCALO-MVP-V2_2026-07-10_F4-activo.json`.

## F5 — Conectar Evolution → n8n y pruebas reales (≈1–1.5h)

- [x] **F5.1** Hecho 2026-07-10: webhook de la instancia `escalo` configurado vía `POST /webhook/set/escalo` → URL pública `https://n8n-o324.srv1815734.hstgr.cloud/webhook/escalo-wa`, evento solo `MESSAGES_UPSERT`, `webhookByEvents: false`. (URL interna no aplica: Traefik en host mode, sin red compartida.)
- [x] **F5.2** Limpieza manual a cargo de Jair (decisión 2026-07-10): borra él mismo en Sheets los contactos `573001112233`, `5491133344455` (duplicado incluido), `5490000000099` y también la fila del test real de F2.3 (`5491146730330`, "Jair chery sower"). Sugerencia: borrar también las filas de esos contactos en `ESCALO_HISTORIAL` para que no arrastren contexto viejo si vuelven a escribir.
- [ ] **F5.3** Correr los casos de prueba obligatorios de `ESCALO_MVP_V2.md` por WhatsApp real entre los teléfonos de Jair. Registrar pasa/falla por caso aquí abajo.

  **Checklist F5.3 (Jair envía cada mensaje desde su otro teléfono; marcar pasa/falla):**

  | # | Mensaje / acción | Resultado esperado | ✅/❌ |
  |---|---|---|---|
  | 1 | `Hola, sigue disponible el Corolla?` | Confirma disponibilidad solo si figura en STOCK; datos reales, sin inventar | |
  | 2 | `Cuánto sale el Cronos?` | Precio exacto del sheet; si no hay precio, no inventa | |
  | 3 | `Tenés Tiggo 2 Pro en blanco?` | Confirma color solo si está en `COLORES DISP.` | |
  | 4 | `Financian el HB20?` | Guion FINANCIACION: pide DNI + anticipo y escala; no inventa planes | |
  | 5 | `Tengo un usado para entregar, me lo toman?` | Guion ENTREGA_USADO: toma sujeta a peritaje, pide datos, escala; `PRECIO DE TOMA` queda vacío | |
  | 6 | `Me hacés descuento por contado?` | No negocia; escala a humano | |
  | 7 | `Quiero ir a ver el Corolla el sábado a las 15hs` | Agenda/deriva visita según flujo; lead CALIENTE | |
  | 8 | 2º mensaje del mismo contacto | Actualiza el lead existente (no duplica fila en LEADS); historial en fila correcta | |
  | 9 | Vehículo inexistente (ej. `Tenés Hilux 2023?`) | No inventa; responde sin coincidencia o escala | |
  | 10 | Usado `EN TALLER` | No promete visita inmediata sin aclarar disponibilidad | |

  Verificar en Sheets tras cada caso: fila en `ESCALO_LEADS` correcta (teléfono solo dígitos), append en `ESCALO_HISTORIAL` con contenido (no filas vacías), `OBSERVACIONES IA` sin nombre repetido.
- [ ] **F5.4** Iterar con MCP hasta que pasen todos. Verificar especialmente: no duplicar leads, historial en fila correcta, no inventar stock/precios, escalados correctos.

## F-SEC — Filtros de seguridad antes de WhatsApp real (doc: `docs/ESCALO_SEGURIDAD_FILTROS.md`)

Workflow desactivado por Jair hasta cerrar esta fase. Implementación 2026-07-10 (madrugada), export `ESCALO-MVP-V2_2026-07-10_seguridad-filtros.json`. Estado de filtros 3/4 en n8n Data Tables: `escalo_pausas` (`2jcm6W3J4D19rwuu`) y `escalo_entrantes` (`EZGQ2DGqmrB3qzVr`); reactivar un contacto a mano = borrar su fila en `escalo_pausas` (UI n8n → Data tables).

- [x] **SEC.1** Filtro origen construido: grupos (`@g.us`), estados (`@broadcast`), newsletters y sin-texto → descarte registrado en HISTORIAL (`ESTADO EJECUCION = DESCARTADO`, `TIPO RESPUESTA = FILTRADO`, `NOTAS LOG = filtro: <motivo>`); eco del propio bot (fromMe por API) → se ignora sin registrar; todo descarte responde al webhook (Evolution ya no espera timeout).
- [x] **SEC.2** Whitelist construida (fail-closed): lee pestaña `ESCALO_WHITELIST`; sin pestaña o vacía = bloquea todo; fila `TODOS` = apagada (producción).
- [x] **SEC.3** Intervención humana construida: fromMe con `source` android/ios → pausa 4h del contacto en `escalo_pausas` + registro `PAUSA_HUMANO`.
- [x] **SEC.4** Límite por contacto construido: debounce 8s (Wait) con agrupado de ráfagas en una consulta (cadena con gap ≤8s) · >10 msg/60s → pausa 30 min · ≥3 contactos pausados por ráfaga en 10 min → alerta WhatsApp a `TELEFONO_ALERTAS` de `ESCALO_NEGOCIO` (si la clave no existe, solo queda el registro).
- [x] **SEC.5** Pestaña `ESCALO_WHITELIST` creada por Jair en el spreadsheet V2: fila 1 `CONTACTO | NOTA`, fila 2 `5491146730330` (número de prueba de Jair). Pendiente opcional: clave `TELEFONO_ALERTAS` en `ESCALO_NEGOCIO`.
- [x] **SEC.6** Probar de a un filtro — 4/4 casos probados: (1) ✅ mensaje en un grupo → sin respuesta + fila DESCARTADO/grupo (probado 2026-07-12 por webhook simulado); (2) ✅ número fuera de la whitelist → sin respuesta + fila DESCARTADO/whitelist; número en la whitelist → responde normal (probado 2026-07-12 por webhook simulado, reconfirmado con tráfico real ejecución `260`); (3) ✅ **confirmado 2026-07-12 con datos reales de producción** (ejecución `261`): un cliente real recibió respuesta manual de Jair desde su iPhone (`fromMe:true`, `source:"ios"`) → el bot detectó `intervencion_humana` y calló 4h con ese contacto (fila en `escalo_pausas`, `hasta_ts - creado_ts` = 4h exactas); (4) ✅ mandar 2-3 mensajes seguidos → una sola respuesta agrupada; ráfaga >10 en 1 min → pausa 30 min registrada (ambos probados 2026-07-12, detalle en Registro).
- [x] **SEC.7** Con los 4 filtros probados: workflow confirmado ACTIVADO por API (`n8n_get_workflow` → `active: true`, `updatedAt` 2026-07-12T12:40 — nunca se desactivó durante SEC.6, no hizo falta reactivar). Sigue **F5.3/F5.4**.

## F6 — Cierre MVP (≈30 min)

- [ ] **F6.1** Verificar 24/7: `restart: always` en todo, workflow activo, revisar ejecuciones con error en n8n. Opcional: `docker restart` de sanidad y confirmar que la sesión de WhatsApp sobrevive.
- [ ] **F6.2** Exportar versión final a `n8n/exports/` (regla del proyecto).
- [ ] **F6.3** Decidir con Jair los post-MVP (abajo) y cuándo pasar al número real de la concesionaria.

---

## Riesgos y decisiones tomadas

- **Baileys es no-oficial** → riesgo bajo pero real de bloqueo del número. Por eso: número de prueba primero; el número real recién cuando el flujo esté estable. Si el negocio escala, evaluar WhatsApp Cloud API oficial.
- **Audios e imágenes:** el MVP los descarta con log. En Argentina se usa mucho el audio → primer candidato post-MVP (transcripción). Alternativa barata intermedia: responder "¿me lo escribís en texto así te ayudo más rápido?" (1 IF + sendText).
- **Mensajes seguidos (ráfagas):** resuelto en F-SEC (2026-07-10): debounce 8s con agrupado + límite >10 msg/60s por contacto, estado en n8n Data Tables (sin tocar Redis).
- **Google Calendar:** credencial existe pero **ningún nodo la usa** todavía. Post-MVP inmediato si Jair lo prioriza (AGENDAR_VISITA hoy escala a humano, lo cual es aceptable para MVP).
- **n8n del VPS en URL hstgr.cloud:** decisión MVP para no tocar el OAuth de Google dos veces. Migrar a `n8n.escalo.com.ar` es cosmético, post-MVP.

## Pendiente — Sistema de códigos de error (anotado 2026-07-13)

Objetivo (pedido de Jair): que cada falla o descarte lleve un **código de etapa** para saber al instante desde qué nodo empezar a debuggear, sin abrir n8n. Formato `E-<área><nº>`, se escribe en `NOTAS LOG` del HISTORIAL y (cuando exista el Bloque C) al inicio de la alerta Telegram/Email. **Solo anotado — implementar cuando Jair lo priorice.**

| Código | Etapa / nodo | Significado |
|---|---|---|
| `E-IN1` | `WA → normalizado` | Payload de Evolution malformado / sin `data` |
| `E-FIL1` | Filtro origen | Descarte grupo/status/newsletter (informativo) |
| `E-FIL2` | Whitelist | Contacto no autorizado (fail-closed) |
| `E-FIL3` | Pausa humana / ráfaga | Contacto en pausa |
| `E-SHEET1` | Leer Stock/Negocio/Historial | Falla de lectura Sheets tras 3 reintentos |
| `E-IA1` | AI Clasificador | Anthropic no responde / timeout |
| `E-IA2` | Validar Salida IA | JSON inválido / intención fuera de lista |
| `E-SHEET2` | Guardar Lead/Historial | Falla de escritura Sheets |
| `E-WA1` | Enviar WhatsApp | Evolution rechaza el envío |
| `E-DEG1` | Rama de degradación | Se activó el fallback de cortesía |

Implementación futura: setear el código en cada rama de error ya existente (Bloque B) y en los descartes de filtros; documentar la tabla canónica en `docs/CODIGOS_ERROR.md`; usarlo como contenido de las alertas del Bloque C.

## Registro de avances

- **2026-07-08** — Plan creado. ESCALO_NEGOCIO completado por Jair. V2 local con E2E OK.
- **2026-07-09** — F0.1 chequeado: zona Cloudflare OK, delegación NIC.ar sin propagar (NXDOMAIN en resolvers públicos). Reorden: avanzar F0.2–F0.4 y F3 mientras propaga. → Próximo: **F0.2**.
- **2026-07-09** — Reorganización docs (aprobada por Jair): CLAUDE.md raíz canónico, addendum a spec V2, docs/CREDENCIALES.md + docs/RUNBOOK.md, git iniciado (commit `1fd877a`), .mcp.json duplicado eliminado, n8n/exports/ creada. Jair debe completar los [COMPLETAR] de CREDENCIALES.md cuando pueda.
- **2026-07-09** — F3.5 confirmado (health check + list workflows OK contra n8n VPS). F4.1–F4.4 aplicados vía MCP en `6IB9N46IpcmpnXmX` (12 operaciones, 14→18 nodos), validado 0 errores, export `ESCALO-MVP-V2_2026-07-09_F4.json`. Pendiente: activar (OK de Jair) + credencial httpHeaderAuth de Evolution cuando exista la API key (F1.1). → Próximo: **F4.5 activar** o **F1** (instalar Evolution; F1.3 espera DNS).
- **2026-07-09** — F0.2 (DNS `evo`), F0.3 (SSH `~/.ssh/escalo_vps`, usuario `root`) y F0.4 (Traefik en `network_mode: host`, certresolver `letsencrypt` vía HTTP-01, URL n8n VPS `https://n8n-o324.srv1815734.hstgr.cloud`) completados. F3.1 (export vía MCP) y F3.2 (import por Jair en n8n VPS, sin activar) completados. → Próximo: **F3.3** (credenciales en n8n VPS).
- **2026-07-09** — F3.3 confirmado (credenciales ya recreadas y asignadas por Jair) y F3.4 corrido y aprobado: MCP ya apunta al n8n del VPS, `n8n_health_check` confirma `apiUrl: https://n8n-o324.srv1815734.hstgr.cloud`. Test de paridad OK (ver detalle en F3.4). Workflow activado solo para el test y desactivado al terminar. → Próximo: **F3.5** (confirmar que la API key del n8n VPS en `.mcp.json` es la definitiva, no una temporal) y luego **F4** (adaptar a Evolution).
- **2026-07-10** — Gran avance (OK global de Jair): F0.1 ✅ (DNS propagó; el A `evo` no existía y Jair lo recreó). F1 completa ✅: Evolution API **v2.3.7** + Postgres + Redis en `/root/evolution/` (secretos en `.env`, 600), HTTPS válido vía Traefik, auth 401 sin key. F2.1 ✅ instancia `escalo` creada (Baileys). F5.1 ✅ webhook `MESSAGES_UPSERT` → n8n público. F4.5 ✅: credencial `Evolution API escalo` creada por API (sin exponer secretos) y asignada, workflow **activado**, regresión (a) paridad OK y (b) WA simulado OK hasta `Enviar WhatsApp` (500 esperado: instancia sin vincular). Fix F4.5: webhooks a `responseMode: responseNode` + rama true cierra en `Responder Webhook`. Export `ESCALO-MVP-V2_2026-07-10_F4-activo.json`. → Próximo: **F2.2** (Jair escanea QR) → F2.3 smoke test → F5.2–F5.4.
- **2026-07-10** — F2.2 ✅ y F2.3 ✅: QR escaneado (`connectionState: open`), Jair mandó un mensaje real por WhatsApp y confirmó haber recibido la respuesta de la IA — pipeline completo (webhook → normalizar → IA → Sheets → Evolution) validado en producción sin intervención manual. De paso surgieron pedidos de mejora de Jair (concurrencia, prompt menos repetitivo/menos "sobre-respuesta", que el escalado a humano no se anuncie al cliente salvo que haga falta, intención nueva para pedidos de trabajo/CV, e intención nueva para redirigir consultas de post-venta/service al `1151321582`) — pendiente de plan y OK antes de tocar el workflow. → Próximo: **F5.2** (limpiar filas de prueba) y evaluar el paquete de mejoras del prompt.
- **2026-07-10 (mediodía)** — Paquete de mejoras aplicado en `6IB9N46IpcmpnXmX`: intenciones nuevas `TRABAJO_CV` y `POSVENTA_SERVICE` con respuesta fija sin escalar, prompt corregido (no sobre-responder, no repetir saludo con historial, no mencionar «asesor»). Validado: 18 nodos, 17 conexiones, 0 errores.
- **2026-07-10 (tarde)** — Comportamiento IA v2 (pedido directo de Jair): preguntar más/informar menos (medios de pago = calificar, no listar) · FINANCIACION pide DNI + anticipo y escala · ENTREGA_USADO informa toma sujeta a peritaje, pide datos del vehículo para tasación del equipo y escala · tono porteño natural anti-bot (fallback «¿En qué más te puedo ayudar?» eliminado) · OBSERVACIONES IA sin fecha, más breve, y registra cambios de vehículo de interés y de temperatura. Detectado workflow **inactivo** tras la sesión anterior → reactivado. Validado 0 errores. Doc canónico nuevo: `docs/COMPORTAMIENTO_IA.md`. Export `ESCALO-MVP-V2_2026-07-10_comportamiento-ia.json`. → Próximo: **F5.2** (limpiar filas de prueba) → F5.3/F5.4.
- **2026-07-10 (tarde 2)** — Regla estricta MEDIOS DE PAGO en el prompt (pedido de Jair: el bot seguía listando todo de golpe): pregunta general → no listar opciones, responder que hay varias formas y preguntar cómo pensaba manejarse (contado/financiado/usado en parte de pago); pregunta puntual («¿aceptan tarjeta?») → confirmar solo ese dato, PROHIBIDO mencionar otros medios. Testeado por webhook de test en 3 turnos (general ✅, tarjeta ✅, transferencia ✅ — hicieron falta 2 iteraciones de endurecimiento). Filas de prueba nuevas con contacto `5490000000099` (entra en F5.2). Doc `docs/COMPORTAMIENTO_IA.md` actualizado. Export `ESCALO-MVP-V2_2026-07-10_medios-pago.json`. → Próximo: **F5.2** (limpiar filas de prueba) → F5.3/F5.4.
- **2026-07-10 (tarde 3)** — F5.2 ✅ resuelto por decisión de Jair: limpieza manual en Sheets a su cargo (contactos `573001112233`, `5491133344455`, `5490000000099` y `5491146730330` — no conserva el lead real del test). Sugerido borrar también esos contactos de `ESCALO_HISTORIAL`. → Próximo: **F5.3** (casos de prueba obligatorios por WhatsApp real).
- **2026-07-10 (noche)** — Fix pre-F5.3 (bug detectado por Jair en WhatsApp real: el bot no encontraba autos que sí están en `ESCALO_STOCK` y llegó a inventar un «Corolla 2022 blanco 45.000 km»): matching de stock por **tokens** de MODELO/MARCA (antes exigía la celda MODELO completa dentro del mensaje) · consulta general de usados/stock → calificar orgánico primero (auto/SUV/camioneta, presupuesto, 0km/usado) y solo si insiste dar LISTADO GENERAL (año, modelo, km si figura, color, incl. «en ingreso»; máx. 20 filas inyectadas al prompt) · prohibición explícita de describir unidades sin coincidencia · OBSERVACIONES IA sin repetir nombre del cliente ni notas duplicadas (recorte en código + regla en prompt). Validado 0 errores, workflow reactivado (estaba inactivo otra vez). Docs: `docs/COMPORTAMIENTO_IA.md`. Export `ESCALO-MVP-V2_2026-07-10_stock-listado.json`. → Próximo: **F5.3** (casos de prueba por WhatsApp real).
- **2026-07-10 (madrugada 11/07)** — **F-SEC implementado** (pedido de Jair: workflow desactivado por precaución, agregar `docs/ESCALO_SEGURIDAD_FILTROS.md` y seguirlo al pie de la letra). Los 4 filtros construidos en `6IB9N46IpcmpnXmX` (18→42 nodos, validado 0 errores, **sigue desactivado**): filtro de origen con registro de descartes y respuesta al webhook (antes los descartes dejaban a Evolution esperando timeout) · whitelist `ESCALO_WHITELIST` fail-closed · pausa 4h por intervención humana · debounce 8s + límite >10/60s + alerta por ataque coordinado. Data Tables creadas: `escalo_pausas`, `escalo_entrantes`. Detalle técnico en el addendum del doc. No se pudo probar en vivo (probar por webhook exige activar y Jair pidió no activar). Corrección de cableado detectada y arreglada en verificación (salida `descartar` del Switch había quedado suelta). Export `ESCALO-MVP-V2_2026-07-10_seguridad-filtros.json`. → Próximo: **SEC.5** (Jair crea `ESCALO_WHITELIST`) → **SEC.6** (pruebas con OK de Jair) → **SEC.7** → F5.3.
- **2026-07-11** — SEC.5 ✅ Jair creó la pestaña `ESCALO_WHITELIST` en el spreadsheet V2: fila 1 `CONTACTO | NOTA`, fila 2 `5491146730330` (su número de prueba). Clave opcional `TELEFONO_ALERTAS` en `ESCALO_NEGOCIO` queda pendiente (no bloqueante). → Próximo: **SEC.6** (probar de a un filtro, requiere OK de Jair para activar el workflow).
- **2026-07-12** — **SEC.6 (3/4)**: revisé el hallazgo pendiente de `onError: continueRegularOutput` en `Leer Whitelist`/`Leer Negocio Alerta`/`Enviar Alerta WhatsApp` rastreando el código downstream real (no solo la config): `Evaluar Whitelist` ya filtra `!r.error`, así que un fallo de lectura de la whitelist cae en lista vacía → `autorizado: false` → sigue fail-closed (SEC.2 intacto); `Msg ráfaga` ni siquiera lee la salida de `Leer Negocio Alerta`/`Enviar Alerta WhatsApp` (reconstruye desde `$('Evaluar Ataque')`), así que un fallo ahí no puede romper ni colgar esa rama. Conclusión: no es un bug de seguridad, no bloqueaba SEC.6. Queda un detalle menor no bloqueante: un fallo real de Sheets en `Leer Whitelist` se loguea igual que "no autorizado" (mismo `motivo: whitelist`), sin distinguirse en el HISTORIAL — mejora de observabilidad opcional para más adelante. Con el OK de Jair (probar 1/2/4 por webhook, caso 3 a cargo de él por WhatsApp real): activé el workflow (42→45 nodos ya contaban desde Bloque A+B) y probé por POST simulado al webhook `/webhook/escalo-wa`: **(1) grupo** → `WA → normalizado` clasificó `ruta:descartar/motivo:grupo`, fila DESCARTADO logueada (HISTORIAL fila 31), sin respuesta ✅. **(2a) número fuera de whitelist** (`5491111111111`) → `descartar/whitelist`, fila DESCARTADO (fila 32), sin respuesta ✅. **(2b) número en whitelist** (`5491146730330`, número de prueba de Jair) → pasó todos los filtros, corrió el pipeline de IA completo y `Enviar WhatsApp` confirmó envío real (Evolution devolvió `status: PENDING`) ✅. **(4a) debounce**: 3 mensajes disparados casi simultáneos → los primeros 2 quedaron en `ignorar/debounce_agrupado`, el 3ro agrupó los 3 textos en una sola consulta y una sola respuesta de la IA (coherente con el conjunto) ✅. **(4b) ráfaga >10/60s**: 12 mensajes concurrentes → detectado `rafaga_12_msgs_60s`, se creó la fila de pausa 30 min en `escalo_pausas` para ese contacto (upsert, sin duplicados) ✅ — la borré enseguida por MCP para no bloquear la prueba manual de Jair (caso 3). Nota aparte (no bloqueante): el ACK que devuelve `Responder Filtros` para los casos descartados que pasan por `Registrar Filtrado` siempre muestra `ruta:"ignorar"` porque ese nodo HTTP sobreescribe `$json` con la respuesta de la API de Sheets — cosmético, no afecta el comportamiento real (Evolution no lee ese body), verificable solo consultando la ejecución o el Sheet directamente. **Workflow queda ACTIVADO** a propósito para que Jair pueda hacer el caso 3 (contestar desde su teléfono a un mensaje del número de whitelist) cuando pueda. → Próximo: Jair prueba caso 3 → cerrar SEC.6 → **SEC.7**.
- **2026-07-11** — Reglas de manejo de errores/alertas documentadas en `CLAUDE.md` (dos capas: por-nodo dentro del flujo vs. catch-all+alertas en flujo separado; Telegram+Email fuera de banda + fallback a Sheet; regla de doc discipline como HQ del SaaS guardada en memoria). Jair había reactivado el workflow tras validar SEC.5; se desactivó de nuevo (con su OK, "si no cambia nada") para trabajar en frío. **Bloque A+B aplicados y validados en `6IB9N46IpcmpnXmX`** (42→45 nodos, 0 errores, 63 conexiones válidas): (A) `retryOnFail` (3/5000ms) en 12 nodos de red (Sheets, HTTP, Anthropic Chat Model). (B) rama de degradación — nueva fila separada del canvas (y=480, debajo del flujo principal) con 3 nodos nuevos (`Fallback Degradación` → `Enviar WhatsApp Cortesía` → `Responder Error Crítico`, ambas salidas de esta última convergen ahí): `onError: continueErrorOutput` + `main[1]` cableado en los 10 nodos del camino crítico (Buscar Lead, Leer Stock, Leer Negocio, Leer Historial, Preparar Contexto, AI Clasificador, Validar Salida IA, Guardar Lead, Guardar Historial, Ensamblar JSON Final) — de paso corrigió `Leer Negocio`/`Leer Historial` que tenían `continueRegularOutput` (antipatrón: error pasaba como dato válido). Además `Enviar WhatsApp` (nodo de éxito) no tenía manejo de error — dejaba el webhook de Evolution colgado si fallaba el envío; ahora su error también cierra en `Responder Webhook` (200, mismo body ya calculado). Verificado con `n8n_get_workflow` (no solo `validate_workflow`, que no detecta el medio-cableado) que las dos mitades de `onError` quedaron en todos los nodos. Bloque C (flujo separado "ESCALO — Alertas": `Error Trigger` → Telegram + Email + `ESCALO_ERRORES`) queda pendiente — requiere de Jair: bot de Telegram (`@BotFather` + `chat_id` vía `@userinfobot`), credencial de envío de mail y pestaña `ESCALO_ERRORES` (`TIMESTAMP | WORKFLOW | NODO | MENSAJE | TELEFONO | EXECUTION_URL`) en el spreadsheet V2. **Hallazgo a revisar en SEC.6** (no tocado, es la cadena de seguridad recién validada): `Leer Whitelist`, `Leer Negocio Alerta` y `Enviar Alerta WhatsApp` tienen `onError: continueRegularOutput` heredado — si falla la lectura, el código aguas abajo puede recibir datos con forma de error y comportarse de forma impredecible (en `Leer Whitelist` esto podría debilitar el fail-closed que es la base de SEC.2). Export `ESCALO-MVP-V2_2026-07-11_manejo-errores.json`. → Próximo: **SEC.6**.
- **2026-07-13** — **Diagnóstico grupos + housekeeping** (pedido de Jair: la IA "lee" mensajes de grupos, cortarlo de raíz). Verificado en el workflow vivo: el filtro de grupos (`WA → normalizado`, `remoteJid.endsWith('@g.us')` → `descartar/grupo`) **funciona** — la IA no responde en grupos ni los manda al clasificador (confirmado ejecución `305`, grupo "RiderJCR" → `descartar/grupo`). PERO Evolution manda `MESSAGES_UPSERT` de todo: cada mensaje de grupo entra a n8n, se lee el texto y se escribe una fila `DESCARTADO` en HISTORIAL → ruido + gasto de ejecuciones. Fix elegido (más óptimo en tiempo/tokens): **cortar en origen** con `groupsIgnore: true` en la instancia `escalo` de Evolution (`POST /settings/set/escalo`) → los grupos ni se emiten, n8n nunca los ve. Config actual verificada: `groupsIgnore: false`. **Pendiente de aplicar** (bloqueado por OK de Jair sobre el write al VPS). Además: anotado el **Sistema de códigos de error** (sección nueva, solo documentado). Housekeeping pendiente de OK: borrar `.git.bak/` (backup viejo redundante), re-exportar workflow (el último export es del 07-11, desfasado), completar sync con GitHub y Drive HQ. → Próximo: aplicar `groupsIgnore` + housekeeping con OK.
- **2026-07-12** — **SEC.6 cerrado (4/4) y SEC.7 cerrado**: verifiqué el caso 3 pendiente sin necesidad de que Jair hiciera una prueba deliberada — el workflow venía activo desde la sesión anterior y ya había tráfico real. Revisé `n8n_executions` de `6IB9N46IpcmpnXmX` y encontré, entre las ~20 ejecuciones de hoy, la ejecución `261` (21:16:39hs): payload real de Evolution con `fromMe:true`, `source:"ios"`, `pushName:"Jair."`, contacto `5492944147534`, mensaje "bueno nos quedamos así y fue, avísame cuando quieras tu alfajor" (claramente real). `WA → normalizado` clasificó `ruta:humano/motivo:intervencion_humana`, `Pausar Contacto (humano)` insertó la fila en `escalo_pausas` (id 5, `hasta_ts - creado_ts` = 14.400.000 ms exactos = 4h) ✅. Caso 3 confirmado con datos de producción, más robusto que una prueba sintética. Con los 4 casos ya probados, cierro **SEC.6**. Para **SEC.7** confirmé por `n8n_get_workflow` (mode minimal) que el workflow sigue `active: true` — nunca se desactivó, no hizo falta reactivar. **Hallazgo adicional revisando `escalo_pausas`** (5 filas, todas `motivo:humano`, todas de hoy): hay 5 contactos reales distintos (no el número de prueba `5491146730330`) que escribieron hoy y fueron atendidos a mano por Jair desde su teléfono — confirmé con la ejecución `260` que al menos uno de ellos (`5492944811677`, "Moth") volvió a escribir después y el bot lo descartó por `motivo:whitelist` (correcto: fail-closed de SEC.2 funcionando, solo el número de prueba de Jair está en `ESCALO_WHITELIST`). No es un bug — es la configuración restrictiva a propósito para probar en frío — pero significa que ninguno de esos 5 leads reales recibió respuesta del bot hoy; Jair los atendió todos manualmente. Se lo dejo planteado para decidir junto con F5.3/F5.4: si ampliar `ESCALO_WHITELIST` antes de los casos de prueba obligatorios, o mantenerla restringida un poco más. → Próximo: **F5.3/F5.4** (casos de prueba obligatorios por WhatsApp real).
