# Proyecto 1 — SmartCity Tech Park

Rediseño de la red del complejo "SmartCity Tech Park" a una topología jerárquica de
**Capa 1 y 2** usando Cisco Packet Tracer: VLANs, VTP, STP y EtherChannel.

| | |
|---|---|
| **Estudiante** | Alex Ricardo Castañeda Rodríguez |
| **Carnet** | 202300476 |
| **Ponderación** | 22 pts |
| **Entrega** | 17/09/2026 |

---

## Parámetros por carnet

| Parámetro | Valor |
|---|---|
| `X` (último dígito) | 6 |
| VLAN Gerencia (Edif. Corporativo) | 16 |
| VLAN Investigacion (Centro I+D) | 26 |
| VLAN Produccion (Planta) | 36 |
| VLAN Servidores (Centro de Datos) | 46 |
| VLAN Visitantes (Edif. Corporativo) | 56 |
| Dominio VTP | Smart_7 |
| Contraseña VTP | proyecto12S2026 |
| EtherChannel | LACP (carnet par) |
| STP | PVST (carnet par) |
| VLAN nativa de trunks | 96 |

---

## Topología (4 áreas)

1. **Centro de Datos (Core)** — switch VTP Server, trunks a los 3 edificios, granja de ≥4 servidores con redundancia.
2. **Centro de I+D** — ≥3 switches interconectados, enlace al Core de mayor ancho de banda, ≥8 PCs.
3. **Edificio Corporativo** — ≥2 alas con su switch de acceso, conectividad entre alas tolerante a fallas, segmento Visitantes aislado con Access Point.
4. **Planta de Producción** — segmento Legacy con Hub (dominio de colisión compartido en Capa 1).

**Seguridad:** banner MOTD `Acceso Restringido - TechPark_202300476` en switches de distribución; VLAN nativa 96 en todos los trunks.

---

## Entregables

- `Proyecto1_202300476.pkt` — topología funcional
- Manual Técnico (`README.md`) con:
  - capturas de topología
  - tabla de dominios de colisión (incluido el del segmento Legacy)
  - tabla de dominios de broadcast (uno por VLAN)
  - evidencias: `show spanning-tree`, `show etherchannel summary`, `show interfaces trunk`, pings OK/fallidos
  - presupuesto estimado de equipos
- Parte física: 2 switches por pareja (VTP Server / Client)

---

## Estado

En construcción.