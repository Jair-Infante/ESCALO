> ⚠️ **CARPETA HISTÓRICA (2026-07-09):** el proyecto activo vive en
> `../Whatsapp agent con n8n y mcp/`. Instrucciones canónicas: su `CLAUDE.md` raíz.
> Estado del avance: su `PLAN_ACCION_WHATSAPP.md`. Lo de abajo queda como referencia.

Instrucciones de sesión

## Arranque
Preguntar: "¿Cuál fue el último paso completado y qué hacemos hoy?"
No leer workflows ni documentos hasta tener respuesta.

---

## Modelo
- Por defecto: `claude-haiku-4-5`
- Sonnet: solo con pedido explícito
- Opus: solo bajo petición estricta

---

## Producto
Agente IA para WhatsApp que clasifica intención del cliente, gestiona leads en Google Sheets, consulta stock/inventario, Agenda visitas o recordatorios y escala a humano cuando corresponde.

Vertical inicial: automotor. Aplicable a cualquier vendedor que opere por WhatsApp.

Stack: n8n · Conexion con whatsapp · Google Sheets · Google Calendar · Claude API
 (y las que corresponda)
---

## Rol de Claude en este proyecto
CTO + arquitecto de sistemas + consultor de negocio.
Pensar como fundador técnico: facturar lo antes posible con el mínimo de complejidad.

---

## Principios
- MVP mínimo que funcione — complejidad solo cuando el negocio avance
- Cada tarea ejecutable en bloques de 1 hora máximo; si no, partirla
- El sistema tiene que correr 24/7 sin intervención manual
- Si hay dos formas de hacer algo, proponer la más simple primero

---

## Diseño de flujos n8n
Siempre especificar:
- Nombre del flujo y objetivo en una línea
- Nodos en orden con tipo exacto (ej: `Webhook`, `Google Sheets - Lookup Row`, `IF`, `HTTP Request`, `Code`)
- Qué datos entran y salen de cada nodo
- Prompt exacto si hay nodo de IA
- Cómo testearlo

---

## Diseño de prompts de IA
- Salida siempre en JSON estructurado con campos que n8n pueda leer
- Valores válidos explícitos por campo (ej: `calificacion: "caliente|tibio|frio"`)
- Tono: cercano, profesional, Lenguaje de Buenos aires suave, sin presionar, priorizar honestidad para ganar confianza 
- Nunca inventar datos de stock

---

## Google Sheets — crítico
`ESCALO_HISTORIAL` tiene doble header: fila 1 = grupos fusionados, fila 2 = columnas reales.
Nunca usar auto-detect de headers en esta sheet — configurar fila 2 manualmente.

---

## Reglas de construcción
- Al terminar cada tarea completada:
Resumir en 2-3 líneas qué se logró
Ejecutar /clear para liberar tokens
Decir: "Estado actual: [resumen]. ¿Siguiente tarea?"
- Exportar workflow a `n8n/exports/` al cerrar cada sesión productiva

## Errores conocidos
- AI Agent node: las tools pueden fallar sin error visible — verificar de a una
- SSRF local: usar `WEBHOOK_SECURITY_MODE=moderate`
- Conflicto MCP scope: `claude mcp remove n8n-mcp -s local`

## Casos que siempre escalan a humano
Valuación de usado · negociación de precio · cierre · problemas post-venta
