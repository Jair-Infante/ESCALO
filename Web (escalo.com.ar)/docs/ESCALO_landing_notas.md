# ESCALO_landing_index.html — notas

## Lo único que falta para publicar

1. **Número de WhatsApp:** abrí el archivo, buscá la línea `const WHATSAPP_NUMBER = "5490000000000"` (cerca del principio, dentro del `<script>`) y poné el tuyo en formato `549` + código de área sin el 0 + número sin el 15. Ese único cambio actualiza los 5 botones de WhatsApp de toda la página.
2. **Kit de marca:** cuando definas los colores, cambiá `--accent` y `--accent-dark` en el bloque `:root` al principio del `<style>`. Toda la web usa esas dos variables, no hay que tocar nada más.
3. **Logo:** hoy dice "ESCALO" en texto. Cuando tengas el logo, reemplazá `<div class="logo">ESCALO</div>` por tu `<img>`.

## Precios reflejados en la página

- **Particulares:** USD 150 instalación (pago único) + USD 50/mes.
- **Empresas:** paquete de 6 líneas, ≈USD 1250 instalación (pago único). La mensualidad por línea todavía no está definida — la dejé como "a coordinar" con un botón directo a WhatsApp en vez de inventar un número. Cuando la definas, actualizamos esa tarjeta en dos minutos.

## Cómo subirla a Hostinger

1. Entrá al hPanel de Hostinger → Administrador de archivos (o por FTP).
2. Andá a la carpeta `public_html` del dominio escalo.com.ar.
3. Subí `ESCALO_landing_index.html` y renombralo a `index.html`.
4. Listo — al entrar a escalo.com.ar ya debería cargar la página.

## Cómo probarla antes de mandarla a nadie

- Abrila en el celu y en la compu.
- Tocá los 5 botones de WhatsApp y confirmá que abren el chat con el número correcto y el mensaje prellenado.
- Mandale el link a 2-3 vendedores conocidos y pedí feedback antes de usarla en una venta real.
