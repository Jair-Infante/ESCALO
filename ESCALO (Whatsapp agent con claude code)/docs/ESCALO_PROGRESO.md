# ESCALO — Progreso de migración a VPS (para retomar en Claude Code)

> ⚠️ **SUPERSEDIDO (2026-07-09):** el plan vivo y actualizado está en
> `Whatsapp agent con n8n y mcp/PLAN_ACCION_WHATSAPP.md`. Usar ese archivo.
> Este documento queda como contexto histórico de la infraestructura.

## Contexto

Se migró la infraestructura de ESCALO de local (Docker en Windows) a un
VPS de Hostinger, para conectar WhatsApp real vía Evolution API. Esto es
adicional al trabajo de ESCALO-MVP-V2 ya documentado en
ESCALO_MVP_V2_MCP_PROMPT.md — ese prompt sigue vigente para la lógica del
workflow. Este documento cubre solo la infraestructura nueva.

---

## Lo que ya está hecho

**Dominio:**
- Comprado: `escalo.com.ar` vía NIC.ar (registrado con CUIT/Clave Fiscal
  de Jair)
- DNS gestionado por **Cloudflare** (no por Hostinger — Hostinger no
  soporta gestión de DNS para TLD `.com.ar`)
- Nameservers delegados en NIC.ar: `alexis.ns.cloudflare.com` y
  `rayne.ns.cloudflare.com`
- A record creado en Cloudflare: `@` → `89.116.225.137`, proxy en **DNS
  only** (nube gris, sin proxy de Cloudflare activado)
- Propagación en curso al momento de este resumen — verificar con antes de asumir que ya
  resuelve

**VPS:**
- Hostinger, plan **KVM2** (2 vCPU, 8GB RAM, 100GB NVMe), pagado en USD
- Hostname: `srv1815734.hstgr.cloud`
- IP pública: `89.116.225.137`
- Sistema operativo: Ubuntu 24.04 LTS
- Hostinger preinstaló automáticamente dos contenedores Docker:
  - `n8n-o324` — instancia de n8n, en ejecución
  - `traefik` — reverse proxy, en ejecución
- Panel de gestión: Hostinger → Administrador de Docker → Proyectos

- Cuenta de Cloudflare creada (plan Free) para gestionar el DNS

## próximos pasos

1. **Confirmar propagación del DNS** — `escalo.com.ar`

2. **Instalar Evolution API**

3. **Migrar el workflow ESCALO-MVP-V2 del n8n local al n8n del VPS**:
   exportar JSON desde n8n local, importar en n8n del VPS.
   ⚠️ Las credenciales NO viajan con el export
   - Google Sheets OAuth2 (la de V2)
   - Google Calendar OAuth2
   - Anthropic (nativa, para el nodo de Agente)

4. **Vincular el número de WhatsApp de prueba en Evolution** 
5. **Modificar el workflow para pulir
6. **Testear con los mismos ~12 casos ya definidos**, esta vez por
   WhatsApp real entre los teléfonos de Jair.
