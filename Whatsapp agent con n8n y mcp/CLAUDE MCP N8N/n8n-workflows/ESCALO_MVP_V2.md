# ESCALO MVP V2 — Prompt optimizado para Claude Code + n8n MCP

> **ADDENDUM 2026-07-09 — leer primero.** Esta spec sigue vigente para reglas de negocio, esquemas de Sheets, intenciones y casos de prueba. Cambios desde su redacción (fuente de verdad del avance: `../../PLAN_ACCION_WHATSAPP.md`):
> - «No conectes WhatsApp real todavía» → **SUPERADO**: WhatsApp real vía Evolution API está aprobado (fases F1–F5 del plan). El webhook de simulación se mantiene para tests.
> - V2 ya está construido y probado E2E: workflow `Rb95ZhoeHWeOAWzp`.
> - Intención nueva `INFO_GENERAL` agregada al enum (responde con la pestaña `ESCALO_NEGOCIO`, pares CLAVE/VALOR).
> - «Guardar Historial» se implementó como HTTP Request a la API de Sheets (append A..R, RAW) por el doble encabezado — el append nativo de n8n escribía filas vacías.
> - Teléfonos normalizados a solo dígitos, sin `+` (evita corrupción de tipo en Sheets y leads duplicados).

## Objetivo

Crear una copia nueva del workflow `ESCALO-MVP-V1` y convertirla en `ESCALO-MVP-V2`, especializado para la concesionaria.

V2 debe usar nodos de **AI Agent dentro de n8n** y herramientas conectadas a Google Sheets y Google Calendar. No debe reconstruirse desde cero si se puede duplicar V1.

## Regla principal

Primero mostrás el plan de nodos. Después, cuando Jair apruebe, creás o modificás el workflow usando n8n MCP.

No conectes WhatsApp real todavía. Entrada por webhook de simulación.

---

## No tocar

No modificar, borrar ni escribir en:

- Workflow V1: `ESCALO-MVP-V1`
- Workflow ID V1: `5CzydNM6GGxtTiFB`
- Spreadsheet V1: `133AKG7qahRvPlW9EetNFAE6KsA0gXhAOlfiZHwl0OMo`

V2 debe usar otra copia de workflow y otra spreadsheet.

---

## Fuente de datos V2

Spreadsheet: `ESCALO-MVPv2-simulacion`  
Spreadsheet ID: `133rSjQRgaBtv7LZMEiPkDUYpS-0tU89Fsv5Qw8HDGLI`

### Pestañas

#### ESCALO_LEADS

Columnas A:M:

`FECHA, NOMBRE, CONTACTO, ESTADO, VEHÍCULO DE INTERÉS, TIPO, COLOR PREFERIDO, MEDIO DE PAGO, VEHÍCULO ENTREGADO, DOMINIO/PATENTE, PRECIO DE TOMA, PRESUPUESTO APROX., OBSERVACIONES IA`

Valores permitidos:

- `ESTADO`: `FRÍO`, `TIBIO`, `CALIENTE`
- `TIPO`: `0KM`, `USADO`
- `MEDIO DE PAGO`: `CONTADO`, `FINANCIADO`, `USADO+CONTADO`, `USADO+FINANCIADO`, `USADO+CONTADO+FINANCIADO`

Regla dura:

- `PRECIO DE TOMA` nunca lo completa la IA.

#### ESCALO_STOCK

Columnas A:L:

`TIPO, MARCA, MODELO, AÑO, DOMINIO/PATENTE, COLOR, CANT. STOCK, COLORES DISP., PRECIO DE VENTA, ESTADO, SUCURSAL, OBSERVACIONES`

Reglas:

- Para `0KM`, confirmar color solo si aparece en `COLORES DISP.`
- Para `USADO`, usar `COLOR`, `DOMINIO/PATENTE`, `PRECIO DE VENTA`, `ESTADO`, `SUCURSAL`
- Si usado está `EN TALLER`, no ofrecer visita inmediata sin aclarar disponibilidad pendiente
- Si no hay coincidencia de stock, no inventar disponibilidad ni precio

#### ESCALO_HISTORIAL

Columnas A:R, encabezado real en **fila 2**:

`TIMESTAMP, TELÉFONO, NOMBRE, CANAL, MENSAJE ORIGINAL, PRODUCTO MENCIONADO, INTENCIÓN, CONFIANZA, REQUIERE ESCALADO, MOTIVO ESCALADO, RESPUESTA ENVIADA, TIPO RESPUESTA, ACCIÓN RECOMENDADA, PRÓXIMO PASO, VENDEDOR ASIGNADO, ESTADO EJECUCIÓN, TIEMPO RESP. (seg), OBSERVACIONES LOG`

Regla crítica:

- Fila 1 no es encabezado.
- Fila 2 es el encabezado real.
- Si el nodo de Google Sheets no permite usar fila 2 como encabezado, usar rango explícito o escribir por posición de columnas para evitar errores de mapeo.

---

## Entrada esperada del webhook

Crear una ruta nueva, por ejemplo:

`/escalo-v2-test`

Payload de prueba:

```json
{
  "contacto": "5491112345678",
  "nombre": "Cliente Test",
  "canal": "webhook-test",
  "mensaje": "quiero ir a ver el Corolla el sábado a las 15hs",
  "timestamp": "{{$now}}"
}
```

Normalizar siempre:

- `contacto`
- `nombre`
- `canal`
- `mensaje`
- `timestamp`

Si falta `timestamp`, usar fecha/hora actual.

---

## Intenciones permitidas

Usar una sola etiqueta:

- `DISPONIBILIDAD`
- `PRECIO`
- `STOCK`
- `PROMOCION`
- `FICHA_TECNICA`
- `FINANCIACION`
- `ENTREGA_USADO`
- `AGENDAR_VISITA`
- `DESCUENTO_NEGOCIACION`
- `RECLAMO`
- `NO_ENTENDIDO`

---

## Criterios de lead

`ESTADO` mide interés de compra, no riesgo.

- `FRÍO`: consulta genérica, sin urgencia ni datos claros
- `TIBIO`: pregunta por stock, precio, color, financiación o modelo concreto
- `CALIENTE`: quiere ir, comprar, reservar, coordinar visita, o avanzar pronto

`REQUIERE ESCALADO` mide si necesita humano.

Marcar `true` si:

- pide descuento o negocia precio
- pregunta financiación específica
- quiere entregar usado
- hay tasación
- hay queja o reclamo
- falta dato confirmado
- intención confusa
- stock/precio/color no confirmado
- quiere cerrar compra/reserva/pago
- la IA no tiene confianza suficiente

Marcar `false` si:

- la consulta es simple
- el dato está confirmado
- no hay compromiso comercial sensible

---

## Arquitectura recomendada para ahorrar tokens

No mandar toda la spreadsheet al agente.

Usar nodos determinísticos antes del AI Agent:

1. Webhook
2. Code/Set: normalizar input
3. Google Sheets: buscar lead por `CONTACTO`
4. Google Sheets: buscar stock por modelo/marca detectada o, si no hay detección previa, pasar solo filas candidatas
5. AI Agent: analizar mensaje con contexto reducido
6. Switch/IF: decidir crear/actualizar lead, agendar o escalar
7. Google Sheets: crear/actualizar lead
8. Google Calendar: crear evento solo si hay fecha/hora clara
9. Google Sheets: escribir historial
10. Respond to Webhook

Usar IA solo para:

- extraer producto/modelo/color/medio de pago
- clasificar intención
- decidir estado del lead
- redactar respuesta segura o resumen para vendedor
- sugerir próximo paso

No usar IA para:

- buscar dentro de toda la planilla si puede hacerlo Google Sheets
- validar enum si puede hacerlo Code node
- completar valores críticos como `PRECIO DE TOMA`
- inventar precio, stock, color o financiación

---

## Nodos de agentes IA

Usar nodos `AI Agent` dentro de n8n, pero mantenerlos controlados.

Modelo por defecto:

`claude-haiku-4-5`

No cambiar a modelo más caro sin avisar.

### Opción preferida: 1 agente compacto

Nombre sugerido:

`AI Agent - Analizar y responder`

Inputs del agente:

- mensaje normalizado
- datos del lead existente, si existe
- filas candidatas de stock, máximo las necesarias
- reglas de escalado
- fecha/hora actual

Outputs estrictos en JSON:

```json
{
  "intencion": "STOCK",
  "confianza": 0.85,
  "producto_mencionado": "Corolla",
  "tipo": "USADO",
  "color_preferido": "",
  "medio_pago": "",
  "vehiculo_entregado": "",
  "dominio_patente": "",
  "presupuesto_aprox": "",
  "estado_lead": "TIBIO",
  "requiere_escalado": false,
  "motivo_escalado": "",
  "quiere_agendar": true,
  "fecha_hora_visita": "2026-07-11T15:00:00-03:00",
  "respuesta_cliente": "Perfecto, podemos coordinar para ver el Corolla el sábado a las 15 hs. Te dejo la visita registrada y un vendedor va a tener el dato a mano.",
  "resumen_vendedor": "",
  "accion_recomendada": "CREAR_EVENTO_CALENDAR",
  "proximo_paso": "Confirmar visita y disponibilidad de la unidad"
}
```

Si no puede responder con seguridad, debe devolver:

- `requiere_escalado: true`
- motivo claro
- resumen para vendedor
- respuesta cliente cautelosa

---

## System prompt compacto para el AI Agent

Usar este prompt dentro del agente:

```text
Sos el agente comercial interno de ESCALO para una concesionaria.

Tarea: analizar un mensaje de cliente, usar SOLO el contexto recibido del workflow, clasificar intención, calificar lead, detectar si requiere humano, y redactar respuesta segura.

Reglas duras:
- No inventes precio, stock, colores, sucursal, financiación, descuento ni disponibilidad.
- PRECIO DE TOMA siempre queda vacío; lo carga un vendedor humano.
- Si falta dato confirmado, escalá.
- Si pide descuento, financiación específica, tasación, reserva, pago, reclamo o cierre de compra, escalá.
- Si quiere visita pero no da fecha/hora clara, pedí fecha/hora; no inventes horario.
- Si usado está EN TALLER, no prometas visita inmediata.
- Para 0KM, confirmá color solo si aparece en COLORES DISP.
- Tono al cliente: cercano, profesional, rioplatense, breve.
- Respondé SOLO JSON válido con las claves pedidas. Sin markdown.
```

---

## Actualización de lead

### Si el contacto ya existe

No crear fila nueva.

Actualizar solo campos nuevos o mejores:

- `ESTADO`
- `VEHÍCULO DE INTERÉS`
- `TIPO`
- `COLOR PREFERIDO`
- `MEDIO DE PAGO`
- `VEHÍCULO ENTREGADO`
- `DOMINIO/PATENTE`
- `PRESUPUESTO APROX.`
- `OBSERVACIONES IA`

Para `OBSERVACIONES IA`:

- no borrar lo anterior
- agregar nueva observación con timestamp breve

Ejemplo:

`[2026-07-05 18:32] Preguntó por Corolla y quiere verlo sábado 15hs.`

### Si el contacto es nuevo

Crear fila en `ESCALO_LEADS`.

Completar solo datos disponibles y confirmados.

Nunca completar `PRECIO DE TOMA`.

---

## Agenda en Google Calendar

Crear evento solo si:

- intención `AGENDAR_VISITA`
- hay vehículo de interés
- hay fecha/hora clara

Evento:

- título: `Visita ESCALO - {nombre} - {vehiculo}`
- descripción: contacto, canal, mensaje original, vehículo, próximo paso
- fecha/hora: la indicada por el cliente
- duración sugerida: 45 minutos

Si no hay fecha/hora clara:

- no crear evento
- respuesta debe pedir día y horario

---

## Historial

Siempre escribir una fila en `ESCALO_HISTORIAL`.

Recordar encabezado real en fila 2.

Campos mínimos:

- `TIMESTAMP`
- `TELÉFONO`
- `NOMBRE`
- `CANAL`
- `MENSAJE ORIGINAL`
- `PRODUCTO MENCIONADO`
- `INTENCIÓN`
- `CONFIANZA`
- `REQUIERE ESCALADO`
- `MOTIVO ESCALADO`
- `RESPUESTA ENVIADA`
- `TIPO RESPUESTA`
- `ACCIÓN RECOMENDADA`
- `PRÓXIMO PASO`
- `VENDEDOR ASIGNADO`
- `ESTADO EJECUCIÓN`
- `TIEMPO RESP. (seg)`
- `OBSERVACIONES LOG`

---

## Respuesta final del webhook

Responder JSON:

```json
{
  "ok": true,
  "workflow": "ESCALO-MVP-V2",
  "lead": {
    "nuevo": true,
    "contacto": "",
    "nombre": "",
    "estado": "",
    "vehiculo_interes": ""
  },
  "analisis": {
    "intencion": "",
    "confianza": 0,
    "requiere_escalado": false,
    "motivo_escalado": ""
  },
  "accion": {
    "lead_actualizado": true,
    "historial_guardado": true,
    "evento_calendar_creado": false,
    "proximo_paso": ""
  },
  "respuesta": {
    "cliente": "",
    "vendedor": ""
  },
  "errores": []
}
```

---

## Casos de prueba obligatorios

Usar modelos del stock actual:

0KM:

- Tiggo 2 Pro
- HB20
- Sandero
- Cronos

Usados:

- Corolla
- Gol Trend

Tests:

1. `Hola, sigue disponible el Corolla?`
2. `Cuánto sale el Cronos?`
3. `Tenés Tiggo 2 Pro en blanco?`
4. `Financian el HB20?`
5. `Tengo un usado para entregar, me lo toman?`
6. `Me hacés descuento por contado?`
7. `Quiero ir a ver el Corolla el sábado a las 15hs`
8. Dos mensajes con el mismo contacto: el segundo debe actualizar el lead, no duplicarlo.
9. Consulta por vehículo inexistente: debe escalar o responder sin inventar.
10. Consulta por usado EN TALLER: no ofrecer visita inmediata sin aclarar.

---

## Antes de construir, responder con este plan

Antes de tocar n8n, mostrar:

1. Si puede duplicar V1 o si conviene crear V2 desde cero
2. Lista de nodos en orden
3. Qué credenciales faltan
4. Cómo resolverá la fila 2 de `ESCALO_HISTORIAL`
5. Cómo evitará duplicar leads
6. Cómo limitará tokens del AI Agent
7. Qué datos pasarán al agente
8. Qué partes serán determinísticas
9. Qué partes usará IA
10. Qué casos de prueba correrá

Después de la aprobación de Jair, construir el workflow.
