# ESCALO — Filtros y protección del flujo (adición a V2)

## Por qué

El flujo fue desactivado por precaución antes de conectarlo a WhatsApp
real. Este documento agrega protecciones al workflow ESCALO-MVP-V2 ya
existente. Todo lo siguiente es aditivo — no tocar la lógica del agente
ya construida.

---

## 1. Filtro de grupos y mensajes de estado

Apenas entra un mensaje, antes de cualquier otro procesamiento: verificar
el origen. Los identificadores de grupo de WhatsApp terminan en `@g.us`
(los individuales en `@s.whatsapp.net`), y los mensajes de Estado suelen
llegar como `status@broadcast` — Evolution API expone esto directamente
en el payload. Si es un grupo o un estado, el flujo se corta ahí mismo:
no se llama a la IA, no se genera respuesta.

## 2. Lista blanca durante la fase de pruebas

Mientras se prueba con WhatsApp real, responder solo a números que Jair
autorizó explícitamente en una lista simple y fácil de editar (una
pestaña de Sheets alcanza). Fuera de esa lista, no hay respuesta
automática. Al pasar a producción, este filtro se apaga (no se elimina)
— ahí cualquiera que escribe es un cliente potencial válido.

## 3. Detección de intervención humana

Si Jair o un vendedor responde personalmente a un cliente desde el
teléfono, el sistema tiene que detectarlo y dejar de responder
automáticamente a ese contacto puntual durante algunas horas, para no
generar una respuesta que contradiga o duplique lo que ya dijo el
vendedor. Pasado ese tiempo, o si se reactiva a mano, el bot vuelve a
responder con normalidad a ese contacto.

## 4. Límite de mensajes por contacto (no un tope diario global)

La protección tiene que ser por número individual, no por volumen total
del día — así el sistema escala sin problema a muchos clientes reales, y
a la vez frena un ataque desde uno o varios números que manden ráfagas de
mensajes de golpe.

- Si el mismo contacto manda varios mensajes seguidos en pocos segundos,
  agruparlos y procesarlos como una sola consulta, no una respuesta por
  mensaje.
- Si un contacto supera un umbral de mensajes en una ventana corta de
  tiempo, ese contacto puntual queda en pausa temporal — el sistema deja
  de responderle por un rato y lo registra. El resto de los contactos
  sigue funcionando con normalidad, sin verse afectado.
- Si varios contactos distintos entran en pausa por este motivo en un
  lapso corto, es señal de ataque coordinado más que de un número
  aislado — ahí sí avisar a Jair, sin frenar el resto del sistema.

Esto es independiente del límite de gasto mensual ya configurado en
Anthropic (USD 50), que sigue como respaldo financiero de última
instancia.

---

## Restricciones

El umbral por contacto tiene que ser generoso — pensado para detectar un
ataque o un error, no para frenar a un cliente real escribiendo rápido.
Todo mensaje descartado por cualquiera de estos filtros conviene que
quede registrado en el historial, con el motivo.

---

## Implementación (addendum técnico — Claude, 2026-07-10)

**Estado: construido y validado (0 errores) en `ESCALO-MVP-V2`
(`6IB9N46IpcmpnXmX`, 18→42 nodos), workflow sigue DESACTIVADO. Pendiente:
pestaña `ESCALO_WHITELIST` (Jair) y pruebas SEC.6 del plan.** Export:
`n8n/exports/ESCALO-MVP-V2_2026-07-10_seguridad-filtros.json`.

Decisiones tomadas al implementar:

- **Filtro 1** vive en `WA → normalizado` (reescrito) + Switch `Filtro origen`.
  Clasifica en: `procesar` / `humano` (fromMe con `source` android/ios) /
  `descartar` (grupo, status/broadcast, newsletter, sin texto, sin remitente)
  / `ignorar` (**eco del propio bot**: fromMe enviado por API — no se
  registra en HISTORIAL para no duplicar cada respuesta; única desviación
  del "todo descarte se registra"). Los descartes se registran en
  `ESCALO_HISTORIAL` (nodo `Registrar Filtrado`, mismo append HTTP A..R RAW)
  con `ESTADO EJECUCION = DESCARTADO` (o `PAUSA_HUMANO`), `TIPO RESPUESTA =
  FILTRADO` y `NOTAS LOG = filtro: <motivo>`; y toda ruta cierra en el nodo
  `Responder Filtros` (antes los descartes dejaban a Evolution esperando
  timeout).
- **Filtro 2 — pestaña `ESCALO_WHITELIST`** (spreadsheet V2, la crea Jair):
  fila 1 = encabezado `CONTACTO | NOTA`. Un número por fila, solo dígitos
  sin `+` (ej. `5491133344455`). Fila especial `TODOS` en CONTACTO =
  whitelist apagada (modo producción). Pestaña vacía o inexistente =
  **bloquear todo** (fail-closed durante pruebas).
- **Filtros 3 y 4 — estado en n8n Data Tables** (no Sheets, para no sumar
  latencia/cuota por mensaje): `escalo_pausas` = `2jcm6W3J4D19rwuu`
  (contacto, hasta_ts, motivo, creado_ts — timestamps en epoch ms) y
  `escalo_entrantes` = `EZGQ2DGqmrB3qzVr` (contacto, ts, mensaje; se
  autolimpia: filas de más de 10 min se borran en cada pasada). Reactivar un
  contacto a mano = borrar su fila en `escalo_pausas` (UI de n8n → Data
  tables).
- **Filtro 3:** pausa de **4 h** por intervención humana. Se detecta con
  `data.source` del payload de Evolution (android/ios = teléfono, humano;
  web/desktop/unknown = API, eco del bot). Verificar en pruebas F-SEC.
- **Filtro 4:** debounce **8 s** (agrupa mensajes del mismo contacto en una
  consulta); umbral generoso **>10 mensajes/60 s** → pausa 30 min; **≥3
  contactos pausados por ráfaga en 10 min** = ataque coordinado → alerta
  WhatsApp al número en clave `TELEFONO_ALERTAS` de `ESCALO_NEGOCIO` (si la
  clave no existe, solo se registra en HISTORIAL).

---

## Orden de implementación

Construir y probar de a uno, verificando cada uno antes de sumar el
siguiente:

1. Filtro de grupos y status
2. Lista blanca de números autorizados
3. Detección de intervención humana
4. Límite de mensajes por contacto

Recién con los cuatro probados, reactivar la conexión con WhatsApp real.
