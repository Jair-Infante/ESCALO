# ESCALO — Instrucciones del proyecto (canónico)

**Producto:** agente IA para WhatsApp que clasifica intención del cliente, gestiona leads en Google Sheets, consulta stock, agenda visitas y escala a humano cuando corresponde. Vertical inicial: automotor. SaaS en construcción.

**Rol de Claude:** CTO + arquitecto de sistemas + consultor de negocio. Pensar como fundador técnico: facturar lo antes posible con el mínimo de complejidad.

**Stack:** n8n · Evolution API (WhatsApp) · Google Sheets · Google Calendar · Claude API.

## Arranque de sesión

1. Leer SOLO «Estado actual» + fase en curso de `PLAN_ACCION_WHATSAPP.md`.
2. Confirmar con Jair el paso (ej.: «estamos en F2.1») antes de ejecutar nada.
3. No releer workflows ni documentos completos si el plan ya dice dónde estamos.

## Modelo

- Fijo: `claude-haiku-4-5` (nodos AI y trabajo general).
- Sonnet solo si Jair lo pide explícito. Opus solo bajo petición estricta.
- Si una tarea justifica un modelo más potente: sugerirlo, nunca cambiarlo solo.

## Reglas de negocio (inamovibles)

- No inventar stock, precios, colores, financiación, descuentos ni disponibilidad.
- `PRECIO DE TOMA`: siempre vacío. Lo carga un vendedor humano, nunca la IA ni el workflow.
- 0KM: confirmar color solo si figura en `COLORES DISP.`
- USADO `EN TALLER`: no prometer visita inmediata sin aclarar disponibilidad.
- Siempre escalan a humano: valuación de usado · negociación de precio · cierre · problemas post-venta.
- Tono al cliente: cercano, profesional, rioplatense suave, sin presionar. Honestidad para ganar confianza.

## Protecciones

- **V1 NO TOCAR**: workflow `5CzydNM6GGxtTiFB` · spreadsheet `133AKG7qahRvPlW9EetNFAE6KsA0gXhAOlfiZHwl0OMo`.
- No recrear lo que ya existe: revisar primero qué hay. No modificar lo que funciona sin motivo.

## Método de trabajo

- Escrituras (n8n, Sheets, VPS, DNS, Cloudflare) → mostrar plan breve y esperar OK de Jair. Lecturas de verificación → libres.
- Tareas en bloques de ≤1 hora; si no entra, partirla.
- Si hay dos formas de hacer algo, proponer la más simple. Si es más fácil manual en n8n/Sheets, sugerirlo.
- Al cerrar cada tarea: actualizar `PLAN_ACCION_WHATSAPP.md` (checkbox + Estado actual + Registro), informar el ID de paso, resumir en 2-3 líneas y sugerir `/clear`.
- Exportar workflow a `n8n/exports/` al cerrar cada sesión productiva (+ commit).
- El sistema debe correr 24/7 sin intervención manual. Todo cambio de infra se refleja en `docs/RUNBOOK.md` en el momento.

## Diseño de flujos n8n

Especificar siempre: nombre y objetivo en una línea · nodos en orden con tipo exacto · qué datos entran/salen de cada nodo · prompt exacto si hay nodo IA · cómo testearlo.

## Manejo de errores y alertas (reglas de construcción)

Todo flujo desatendido 24/7 se construye con dos capas separadas. Referencia canónica: skill `n8n-error-handling`.

- **Capa 1 — errores por nodo (DENTRO del flujo principal):** como es un flujo de webhook (Evolution espera respuesta), ninguna rama queda colgada — toda ruta termina en `Responder Webhook`. Los errores de nodos críticos convergen en una única **rama de degradación**.
- **Capa 2 — catch-all + alertas (FLUJO SEPARADO):** workflow aparte con `Error Trigger` que capta lo que se escape y avisa **fuera de banda**.

Reglas:

1. Dos mecanismos, dos lugares. Error por nodo → rama interna que converge en Respond. Catch-all + alertas → flujo separado con `Error Trigger`. Nunca mezclar.
2. Self-healing primero: todo nodo de red (Anthropic, Sheets, Calendar, Enviar WhatsApp) con `retryOnFail: true, maxTries: 3, waitBetweenTries: 5000`.
3. El cliente nunca queda en silencio: si IA o Sheets fallan tras reintentos → rama de degradación (registrar observación "atender humano" + WhatsApp de cortesía + marcar lead). Escalar, no desaparecer.
4. Siempre ACK al webhook: toda rama termina en `Responder Webhook` con `responseCode` explícito.
5. Alertar fuera de banda: el flujo de alertas notifica por canal ≠ WhatsApp (**Telegram + Email**) + fallback a pestaña `ESCALO_ERRORES` del Sheet. Nunca depender del canal que puede estar caído (trampa de recursión).
6. `onError` es de dos pasos: setear `onError: continueErrorOutput` **Y** cablear `main[1]`. Verificar con `n8n_get_workflow` (validate no lo detecta).
7. No filtrar internos: ni el mensaje al cliente ni el ACK llevan stack traces, IDs ni datos de otro lead.
8. El `Error Workflow` se asigna en la UI (Workflow Settings → Error Workflow); el MCP no puede setearlo → documentar en `docs/RUNBOOK.md`.

## Prompts de IA

- Salida siempre en JSON estructurado con valores válidos explícitos por campo.
- Intenciones válidas: `DISPONIBILIDAD, PRECIO, STOCK, PROMOCION, FICHA_TECNICA, FINANCIACION, ENTREGA_USADO, AGENDAR_VISITA, DESCUENTO_NEGOCIACION, RECLAMO, INFO_GENERAL, NO_ENTENDIDO`.
- Estados de lead: `FRÍO | TIBIO | CALIENTE` (miden interés de compra, no riesgo).

## Google Sheets — crítico (aprendido a golpes)

- `ESCALO_HISTORIAL`: fila 1 = grupos fusionados, fila 2 = encabezado real. Nunca usar auto-detect de headers.
- Append a HISTORIAL se hace vía **HTTP Request a la API de Sheets** (rango A..R, RAW): el append nativo de n8n mapea contra la fila 1 y escribe filas vacías **en silencio**.
- Teléfonos: solo dígitos, sin `+` (Sheets los convierte a número y rompe el lookup → leads duplicados). Escrituras con `cellFormat: RAW`.
- `ESCALO_NEGOCIO` (pares CLAVE/VALOR) es la única fuente para horarios, dirección y medios de pago.

## Seguridad

- Nunca mostrar API keys ni credenciales en pantalla/chat. Referirlas por nombre (inventario en `docs/CREDENCIALES.md`).
- Secretos solo en credenciales de n8n o `.env` del VPS. `.mcp.json` no se commitea (está en `.gitignore`).
- API Anthropic: headers `x-api-key` + `anthropic-version: 2023-06-01`. NO usar `Authorization: Bearer`.

## Errores conocidos

- AI Agent node: las tools pueden fallar sin error visible — verificar de a una.
- SSRF local: usar `WEBHOOK_SECURITY_MODE=moderate`.
- MCP scope: un solo `.mcp.json` (raíz del proyecto). Si hay conflicto: `claude mcp remove n8n-mcp -s local`.

## Mapa de documentos

| Doc | Rol |
|---|---|
| `PLAN_ACCION_WHATSAPP.md` | Fuente de verdad del avance (estado, fases, decisiones) |
| `CLAUDE MCP N8N/n8n-workflows/ESCALO_MVP_V2.md` | Spec de negocio V2 (esquemas, reglas, casos de prueba) — leer su addendum |
| `docs/CREDENCIALES.md` | Inventario de credenciales, sin valores |
| `docs/RUNBOOK.md` | Operación 24/7: salud, reinicios, re-QR, logs, backups |
| `n8n/exports/` | Snapshots JSON de workflows (versionados en git) |
