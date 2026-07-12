# Plan de escalo.com.ar — Landing de una página

## 1. Diagnóstico honesto

Hoy ESCALO tiene 0 clientes piloto y 0 clientes pagando (según el estado del proyecto). Pediste que la web "refuerce casos de éxito" — pero no hay casos de éxito todavía. Si construimos esa sección ahora con contenido inventado, hay que rehacerla apenas cierres tu primer cliente, y un testimonio falso te puede costar credibilidad en un rubro chico como el automotor argentino, donde todos se conocen.

**Ajuste:** dejamos el espacio para prueba social diseñado y listo, pero lo llenamos con lo que sí es verdad hoy (tu experiencia como vendedor, el problema que ves todos los días, cómo funciona el sistema). Cuando tengas el primer piloto, actualizás esa sección en 10 minutos.

## 2. Oportunidad

La web no tiene que traer tráfico frío — vos vendés cara a cara a gente que conocés. Su trabajo es cerrar la venta después de la conversación: el prospecto la ve, confirma que sos algo serio, y te escribe por WhatsApp. Landing corta, un solo objetivo: WhatsApp.

## 3. Punto ciego a cuidar

No sumar precios, features o casos que no estén confirmados. Todo lo que diga la web tiene que poder sostenerse en una llamada de venta sin que te agarren en un invento.

## 4. Solución simple: landing de una página, seccionada

Una sola página HTML, mobile-first (tus prospectos la van a abrir desde el celu), con estas secciones en orden:

| # | Sección | Contenido |
|---|---------|-----------|
| 1 | Hero | Frase de una línea + botón "Hablar por WhatsApp" |
| 2 | Problema | 3 dolores típicos del vendedor de autos sin sistema (leads que se enfrían, seguimiento manual, horario limitado) |
| 3 | Cómo funciona | 3-4 pasos simples: cliente escribe por WhatsApp → IA responde y clasifica → CRM se actualiza solo → vos recibís el lead caliente |
| 4 | Producto | Copiloto WhatsApp + CRM automático en Sheets, explicado sin jerga técnica |
| 5 | Precios | Setup + mensual, en ARS y USD |
| 6 | Por qué confiar | Tu historia como vendedor (no testimonios inventados) — reemplaza "casos de éxito" hasta tener el primero real |
| 7 | FAQ | 4-5 objeciones típicas (¿se banea el WhatsApp?, ¿reemplaza al vendedor?, ¿cuánto tarda en andar?) |
| 8 | CTA final | Repetir botón de WhatsApp |

## 5. Herramientas

- **Código:** HTML + CSS + JS vanilla (sin frameworks pesados). Una sola página, liviana, rápida.
- **Hosting:** Hostinger, plan de hosting compartido — no hace falta el VPS que vas a usar para n8n. Es otro servicio dentro de la misma cuenta.
- **Dominio:** escalo.com.ar apuntando al hosting.
- Nada de pago adicional más allá de lo que ya tenés pensado gastar en Hostinger.

## 6. Bloques de trabajo (1-2h cada uno)

- [ ] Bloque 1: estructura HTML + Hero + Problema
- [ ] Bloque 2: Cómo funciona + Producto + Precios
- [ ] Bloque 3: Por qué confiar + FAQ + CTA final
- [ ] Bloque 4: responsive (mobile primero) + botón WhatsApp con mensaje prellenado
- [ ] Bloque 5: deploy en Hostinger + apuntar dominio

## 7. Datos que necesito de vos para escribir el código

1. Tagline / frase del Hero (una línea que resuma qué hace ESCALO)
2. Precio final decidido (setup y mensual, ARS y USD)
3. Número de WhatsApp de contacto
4. Color de marca (o decimos uno: sugiero un color oscuro tipo azul petróleo + acento, transmite "tech serio" sin ser genérico)
5. Logo si tenés, si no arrancamos solo con texto "ESCALO"

## 8. Cómo probarlo

- Abrir en el celu y en la compu antes de mandarla a nadie
- Verificar que el botón de WhatsApp abra el chat con el número correcto y un mensaje prellenado tipo "Hola, vi la web de ESCALO y quiero saber más"
- Mandar el link a 2-3 vendedores que conozcas y pedir feedback honesto antes de usarla en una venta real

## 9. Métrica que importa

No importa el tráfico total. Importa una sola cosa: **clicks en el botón de WhatsApp**. Se puede trackear simple con Google Analytics o incluso contando conversaciones nuevas que digan "vi la web".

## 10. Cómo se convierte en vendible

El link va en tu firma de WhatsApp Business y se lo mandás a cada prospecto antes de la llamada. Refuerza que sos algo serio, no reemplaza la venta que ya hacés vos.

## 11. Acción concreta para hoy

Mandame los 5 datos del punto 7 (tagline, precio final, WhatsApp, color, logo) y arranco el Bloque 1 del código.
