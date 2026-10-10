# Informe de desarrollo - Práctica 2

Carnet: **202300476**. Simulador: Cisco Packet Tracer 9.0.1. Archivo: [`Practica2_202300476.pkt`](../packet-tracer/Practica2_202300476.pkt).

## Resultado

Red de capa 2 con **cinco zonas, diez switches y 30 PCs** (seis por zona), unidos por 43 enlaces Ethernet. Funcionan las VLANs 16/26/36/46/56, la nativa 99, la Blackhole 999, VTP v2, Rapid PVST+ con un root por zona y un EtherChannel LACP entre las dos zonas críticas. Los pings dentro de cada VLAN responden y los pings hacia otras VLANs fallan, como corresponde a una red sin capa 3.

![Topología final con 30 PCs](evidencias/capturas/03_etapa3_topologia_final_30pcs.png)

Todas las imágenes de este informe son capturas de ventanas completas de Packet Tracer. El índice está en [`evidencias/README.md`](evidencias/README.md).

## 1. Planificación

1. Se identificaron los cinco establecimientos en el directorio de Cayalá y se estimó el rol, el tráfico y la criticidad de cada zona (sección 2 del [manual técnico](Manual_Tecnico.md)).
2. Se dimensionaron las subredes con VLSM dentro de 10.76.0.0/24, de la mayor demanda a la menor: Z1 /26, Z4 /26, Z2 /27, Z3 /28 y Z5 /28. En cada subred, la primera IP útil queda reservada como gateway.
3. Las VLANs se numeraron con el último dígito del carnet: 16, 26, 36, 46 y 56. La 99 es la nativa y la 999 la Blackhole.
4. Como el carnet termina en número par, el EtherChannel usa LACP.
5. El auxiliar aclaró en el foro que no se implementa capa 3. Por eso no hay routers, SVIs con IP ni `ip routing`.

## 2. Evolución del diseño

| Etapa | Captura |
|---|---|
| 1. Primera red con los 157 hosts de la demanda proyectada y 15 switches | ![Red completa](evidencias/capturas/01_etapa1_red_completa_157pcs.png) |
| 2. Modelo compacto con 3 PCs por zona, para que el diagrama se lea con claridad | ![Compacta](evidencias/capturas/02_etapa2_compacta_15pcs.png) |
| 3. Modelo final con 6 PCs por zona, zonas dibujadas, leyenda y sin cables cruzados | ![Final](evidencias/capturas/03_etapa3_topologia_final_30pcs.png) |

La red de 157 PCs era difícil de leer y no aportaba nada a la demostración de los protocolos. Por eso se redujo a un modelo compacto. Las cantidades 60/28/12/50/7 se siguen usando para calcular las subredes.

## 3. Construcción de la topología

1. Cada zona tiene un switch principal 3560-24PS y uno secundario 2960-24TT.
2. Los PCs se conectan con cable directo: PC 01 y PC 04 al principal (Fa0/10 y Fa0/11), y PC 02, 03, 05 y 06 al secundario (Fa0/2 a Fa0/5). Así, todo ping entre un PC del principal y uno del secundario cruza un trunk.
3. Los switches se unen con cable cruzado:
   - **Triángulo** entre los principales de Z1, Z2 y Z4. El lado Z1-Z4 son dos enlaces Gigabit que forman Po1.
   - **Ramales en estrella** de Z3 y Z5, que cuelgan de SW-Z2-P.
   - **Respaldos cruzados:** cada secundario de Z1 y Z4 tiene un segundo uplink hacia el principal de la otra zona crítica.
4. Se intentó un enlace de fibra entre Z2 y Z3, pero el 3560 no tiene puertos ópticos en este simulador. Se usó cobre Gigabit.

## 4. Configuración por CLI

Todos los switches se configuraron solo por CLI. Los comandos completos de cada uno están en [`scripts/`](scripts/). Se aplicaron en este orden:

1. **Acceso al switch:** `hostname`, `enable secret`, contraseña y `exec-timeout` en la consola, VTY cerradas, `service password-encryption` y banner.
2. **VTP:** dominio `202300476`, versión 2 y contraseña. Los cinco principales son Server y los cinco secundarios Client.
3. **VLANs:** se crearon las siete VLANs en SW-Z1-P y los demás switches las aprendieron por VTP.
4. **Trunks:** 802.1Q con nativa 99, `nonegotiate` y lista explícita de VLANs. Una VLAN solo viaja por un trunk si detrás hay hosts de esa VLAN o un camino alternativo hacia ellos. El triángulo lleva `1,16,46,99` y los ramales y accesos `1,X6,99`.
5. **Rapid PVST+:** cada principal es root de su VLAN con prioridad 4096, y el respaldo usa 8192.
6. **Puertos de PC:** modo access en la VLAN de la zona, con PortFast y BPDU Guard.
7. **Puertos sin uso:** VLAN 999, descripción `UNUSED_BLACKHOLE` y `shutdown`.
8. **EtherChannel:** `channel-group 1 mode active` en Gi0/1 y Gi0/2 de SW-Z1-P y SW-Z4-P. Los miembros y Po1 tienen los mismos parámetros de trunk.
9. **PCs:** IP, máscara y gateway reservado según la [tabla de hosts](Manual_Tecnico.md#hosts-instalados).
10. `write memory` en los diez switches.

Ejemplo: bloque de interfaces de SW-Z5-A1 aplicado por CLI y el `running-config` resultante.

![Configuración por CLI en SW-Z5-A1](evidencias/capturas/04_config_por_cli_SW-Z5-A1.png)

![running-config de SW-Z5-A1](evidencias/capturas/05_running_config_SW-Z5-A1.png)

## 5. Diseño visual

- Z2 queda en el centro de la fila superior como núcleo, con los ramales Z3 y Z5 a cada lado.
- Abajo, Z4 a la izquierda y Z1 a la derecha, unidas por Po1 en horizontal. Un respaldo cruzado pasa por encima de Po1 y el otro por debajo, así que ningún cable se cruza.
- Cada zona tiene fondo pastel, borde de su color y un rótulo con VLAN, subred, criticidad y rol del principal.
- Los enlaces entre switches muestran sus puertos.
- La leyenda explica colores, tipos de cable y estados de puerto, y los paneles inferiores resumen VLANs, trunks y protocolos.

Se comprobó geométricamente que ningún cable corta a otro ni pasa sobre un dispositivo o una etiqueta.

## 6. Pruebas

### VTP

Los cinco principales están en modo Server y los secundarios en Client. Todos tienen el dominio `202300476`, la versión 2, 12 VLANs y la misma revisión (160).

![VTP servidor SW-Z1-P](evidencias/capturas/10_vtp_servidor_SW-Z1-P.png)

![VTP servidor SW-Z2-P](evidencias/capturas/11_vtp_servidor_SW-Z2-P.png)

![VTP servidor SW-Z4-P](evidencias/capturas/12_vtp_servidor_SW-Z4-P.png)

![VTP cliente SW-Z1-A1](evidencias/capturas/13_vtp_cliente_SW-Z1-A1.png)

### VLANs y Blackhole

En SW-Z1-P, la VLAN 16 tiene Fa0/10 y Fa0/11, y el resto de puertos de acceso está en la VLAN 999. En SW-Z1-A1, los puertos sin uso aparecen como `UNUSED_BLACKHOLE`, `disabled`, en la VLAN 999. En total hay 204 puertos libres, todos en VLAN 999 y apagados.

![show vlan brief SW-Z1-P](evidencias/capturas/20_vlan_brief_SW-Z1-P.png)

![show vlan brief SW-Z1-A1](evidencias/capturas/21_vlan_brief_SW-Z1-A1.png)

![Puertos Blackhole SW-Z1-A1](evidencias/capturas/22_puertos_blackhole_SW-Z1-A1.png)

### Trunks

Ningún trunk permite `all` y todos usan la VLAN nativa 99. El triángulo Z1–Z2–Z4 lleva `1,16,46,99`, y los ramales y los accesos llevan solo la VLAN de su zona: `1,X6,99`.

![Trunks SW-Z1-P](evidencias/capturas/30_trunks_SW-Z1-P.png)

![Trunks SW-Z1-A1](evidencias/capturas/31_trunks_SW-Z1-A1.png)

![Trunks SW-Z2-P (núcleo)](evidencias/capturas/32_trunks_SW-Z2-P.png)

![Trunks SW-Z3-P](evidencias/capturas/33_trunks_SW-Z3-P.png)

![Trunks SW-Z4-P](evidencias/capturas/34_trunks_SW-Z4-P.png)

![Trunks SW-Z5-P](evidencias/capturas/35_trunks_SW-Z5-P.png)

### Rapid PVST+ y root bridge

Cada principal muestra `This bridge is the root` para su VLAN, con prioridad 4096 más el ID de la VLAN (4112, 4122, 4132, 4142 y 4152). En SW-Z1-A1, Gi0/1 es Root FWD y Gi0/2, el respaldo hacia Z4, está en Altn BLK.

![Root VLAN 16](evidencias/capturas/40_stp_root_vlan16_SW-Z1-P.png)

![Root VLAN 26](evidencias/capturas/41_stp_root_vlan26_SW-Z2-P.png)

![Root VLAN 36](evidencias/capturas/42_stp_root_vlan36_SW-Z3-P.png)

![Root VLAN 46](evidencias/capturas/43_stp_root_vlan46_SW-Z4-P.png)

![Root VLAN 56](evidencias/capturas/44_stp_root_vlan56_SW-Z5-P.png)

![VLAN 16 en SW-Z1-A1](evidencias/capturas/45_stp_vlan16_SW-Z1-A1.png)

![VLAN 16 en SW-Z2-P](evidencias/capturas/46_stp_vlan16_SW-Z2-P.png)

![Resumen STP SW-Z1-P](evidencias/capturas/47_stp_summary_SW-Z1-P.png)

### EtherChannel LACP

En los dos extremos aparece `Po1(SU) LACP Gig0/1(P) Gig0/2(P)`.

![EtherChannel SW-Z1-P](evidencias/capturas/50_etherchannel_summary_SW-Z1-P.png)

![Detalle Po1 SW-Z1-P](evidencias/capturas/51_etherchannel_port-channel_SW-Z1-P.png)

![EtherChannel SW-Z4-P](evidencias/capturas/52_etherchannel_summary_SW-Z4-P.png)

### show ip route

El switch no tiene gateway predeterminado y la salida muestra su caché de redirecciones, no rutas entre VLANs. Es lo esperado: no hay `ip routing`.

![show ip route en SW-Z1-P](evidencias/capturas/55_show_ip_route_SW-Z1-P.png)

### Pings desde los PCs

Se hicieron 10 pings dentro de la VLAN (2 por zona), todos con 4/4, y 6 pings hacia otra VLAN, todos con 0/4. Ocho de los pings dentro de la VLAN cruzan el trunk entre el principal y el secundario. Los otros dos son entre PCs del mismo switch: PC-Z2-04 → 10.76.0.130 (ambos en SW-Z2-P) y PC-Z3-05 → 10.76.0.167 (ambos en SW-Z3-A1).

| Origen | Misma VLAN (4/4) | Otra VLAN (0/4) |
|---|---|---|
| PC-Z1-04 | 10.76.0.3, 10.76.0.6 | 10.76.0.130, 10.76.0.66 |
| PC-Z2-04 | 10.76.0.130, 10.76.0.135 | 10.76.0.2 |
| PC-Z3-05 | 10.76.0.162, 10.76.0.167 | 10.76.0.178 |
| PC-Z4-01 | 10.76.0.67, 10.76.0.70 | 10.76.0.162 |
| PC-Z5-01 | 10.76.0.179, 10.76.0.183 | 10.76.0.66 |

![IP de PC-Z1-01](evidencias/capturas/60_ip_config_PC-Z1-01.png)

![ipconfig y pings PC-Z1-04](evidencias/capturas/61_ipconfig_y_pings_PC-Z1-04.png)

![Pings PC-Z2-04](evidencias/capturas/62_pings_PC-Z2-04.png)

![Pings PC-Z3-05](evidencias/capturas/63_pings_PC-Z3-05.png)

![Pings PC-Z4-01](evidencias/capturas/64_pings_PC-Z4-01.png)

![Pings PC-Z5-01](evidencias/capturas/65_pings_PC-Z5-01.png)

![Cuatro Command Prompt a la vez](evidencias/capturas/66_pings_cuatro_zonas_en_paralelo.png)

Un ping fallido no basta, por sí solo, para demostrar la segmentación. Lo que la demuestra es el conjunto: los pings dentro de cada VLAN responden, los pings hacia otras VLANs fallan y la configuración no tiene routing.

### Simulación de ARP e ICMP

En modo Simulation, con los filtros en ARP e ICMP, PC-Z3-05 hizo ping a 10.76.0.164 (PC-Z3-03), una IP que no estaba en su caché ARP.

- SW-Z3-A1 recibe el broadcast ARP por Fa0/4 y lo reenvía solo por los puertos de la VLAN 36: Fa0/2, Fa0/3, Fa0/5 y Gi0/1.
- SW-Z3-P lo reenvía por sus tres puertos de la VLAN 36 en forwarding: Fa0/10, Fa0/11 y Gi0/1 hacia SW-Z2-P. En la lista de eventos, la columna *Last Device* lo muestra.
- Ningún evento viene de SW-Z2-P, así que el broadcast se queda en el núcleo: sus otros trunks no llevan la VLAN 36.
- Después de la respuesta ARP, el Echo Request y el Echo Reply completan el ciclo.

![ARP en curso](evidencias/capturas/70_simulacion_arp_en_curso_Z3.png)

![Lista de eventos ARP/ICMP](evidencias/capturas/71_simulacion_arp_icmp_completo_Z3.png)

![Detalle PDU ARP en SW-Z3-A1](evidencias/capturas/72_pdu_arp_SW-Z3-A1.png)

### Pruebas de fallo

1. Se apagó `Gi0/1` de SW-Z1-P, que es un miembro de Po1. El canal siguió en `Po1(SU)`, con `Gig0/1(D)` y `Gig0/2(P)`.
2. Con ese miembro todavía caído, se apagó `Fa0/1`, el uplink de SW-Z1-A1. SW-Z1-A1 tomó Gi0/2 como Root FWD (costo 23), es decir, el camino Z1-A1 → Z4-P → Po1 → Z1-P.
3. PC-Z1-04 hizo ping a 10.76.0.3 y 10.76.0.7, conectados a SW-Z1-A1. Los dos respondieron 4/4 por el camino de respaldo.
4. Se restauró con `no shutdown`: Po1 volvió a tener los dos miembros en `(P)` y los trunks quedaron igual que antes.

También se apagaron Po1 y los uplinks de acceso de Z1 y Z4 al mismo tiempo. Las VLANs 16 y 46 siguieron respondiendo 4/4 a través de SW-Z2-P.

![Fallo de un miembro de Po1](evidencias/capturas/80_fallo_miembro_po1_SW-Z1-P.png)

![Uplink apagado en SW-Z1-P](evidencias/capturas/81_fallo_uplink_y_stp_SW-Z1-P.png)

![SW-Z1-A1 usa Gi0/2](evidencias/capturas/82_fallo_stp_root_por_gi0-2_SW-Z1-A1.png)

![Ping por el respaldo](evidencias/capturas/83_fallo_ping_por_respaldo_PC-Z1-04.png)

![Pantalla completa con los enlaces caídos](evidencias/capturas/84_fallo_pantalla_completa_enlaces_caidos.png)

![Restauración](evidencias/capturas/85_restauracion_SW-Z1-P.png)

## 7. Problemas encontrados y soluciones

| Problema | Diagnóstico y solución |
|---|---|
| El enlace de fibra Z2-Z3 no se creaba | El 3560 no tiene puertos ópticos en Packet Tracer; se usó cobre Gigabit |
| Pérdidas de pings y BPDUs reenviados al quitar VLAN 1 de los trunks | En modo Simulation se vio que los 3560 no entregaban esos BPDUs al proceso STP. Permitir VLAN 1 en los trunks, con la 99 como nativa, eliminó el fallo |
| Miembros de LACP suspendidos o independientes | Los miembros y Po1 tenían parámetros de trunk distintos. Se igualaron y los dos quedaron `(P)` |
| Al abrir el archivo, los switches aparecían en VTP v1 | Se volvió a aplicar `vtp version 2` en los servidores y se guardó; la versión 2 se mantiene |
| La revisión VTP sube al abrir el archivo | Es normal en Packet Tracer; lo importante es que coincida en los diez switches |
| Puertos de PC en LRN y pings perdidos justo después de abrir | La red aún no había convergido. Con Fast Forward Time converge y los pings responden 4/4 |
| `terminal length 0` no existe en esta imagen IOS | Las salidas largas se recorren página por página con `--More--` |
| `show lacp neighbor` no está implementado | El estado del canal se comprueba con `show etherchannel summary` |

## 8. Correspondencia con la hoja de calificación

| Criterio | Evidencia o documento |
|---|---|
| Topología híbrida, cinco zonas y cableado | Captura de la topología y secciones 2-5 del manual |
| VLANs, nativa, Blackhole y trunks | Scripts CLI, `show vlan brief` y `show interfaces trunk` |
| VTP | Cinco Server y cinco Client, mismo dominio, versión y revisión |
| Rapid PVST+ y root por zona | Prioridades en los scripts y salidas de los cinco roots |
| LACP y conectividad | `Po1(SU)` con dos miembros, pings y pruebas de fallo |
| Manual, informe y scripts | [Manual técnico](Manual_Tecnico.md), este informe y [`scripts/`](scripts/) |
| Presentación | Zonas dibujadas, leyenda, nombres consistentes y cero cruces de cables |
| Comprensión teórico-práctica | Simulación ARP/ICMP en modo Simulation |

Las salidas `show` y el `running-config` de los diez switches están en [`evidencias/salidas_switches/`](evidencias/salidas_switches/). Para revisar en vivo, abrir el `.pkt`, esperar a que converja (o pulsar Fast Forward Time) y repetir los comandos y pings.
