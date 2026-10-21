# Proyecto 2 — Infraestructura Financiera y Alta Disponibilidad BanTech GT

**Manual Técnico**

Diseño e implementación en Cisco Packet Tracer de la red nacional de *BanTech GT*. La red tiene un
backbone redundante que une cuatro dominios de enrutamiento (OSPF, EIGRP, RIPv2 y rutas estáticas)
y cuatro sedes diseñadas desde cero. En las sedes se aplicaron VLANs con VLSM, enrutamiento
inter-VLAN, VTP, Rapid PVST+, EtherChannel LACP y HSRP.

| | |
|---|---|
| **Estudiante** | Alex Ricardo Castañeda Rodríguez |
| **Carné** | 202300476 |
| **Curso** | Redes de Computadoras 1 — Segundo Semestre 2026 |
| **Grupo** | 2 (coordinador) |
| **Ponderación** | 33 pts |
| **Archivo de topología** | [`packet-tracer/Proyecto2_202300476.pkt`](packet-tracer/Proyecto2_202300476.pkt) |
| **Versión de Packet Tracer** | 9.0.1 |

> Todas las salidas de comandos, tablas de rutas y resultados de ping de este manual se tomaron de la
> topología en ejecución. Las capturas completas están en [`docs/img/evidencias/`](docs/img/evidencias/)
> y el texto de cada salida está en [`docs/evidencias/salidas/`](docs/evidencias/salidas/).

---

## Índice

1. [Parámetros de identificación](#1-parámetros-de-identificación)
2. [Resumen de la solución](#2-resumen-de-la-solución)
3. [Topología](#3-topología)
4. [Inventario de equipos](#4-inventario-de-equipos)
5. [Backbone nacional](#5-backbone-nacional)
6. [Direccionamiento IP y subneteo](#6-direccionamiento-ip-y-subneteo)
7. [Tabla de VLANs](#7-tabla-de-vlans)
8. [Sede Occidente — Agencia Regional Comercial](#8-sede-occidente--agencia-regional-comercial)
9. [Sede Norte — Centro de Autorización de Créditos](#9-sede-norte--centro-de-autorización-de-créditos)
10. [Sede Oriente — Centro Financiero Legado](#10-sede-oriente--centro-financiero-legado)
11. [Data Center y Sede Central](#11-data-center-y-sede-central)
12. [Configuración común a todos los equipos](#12-configuración-común-a-todos-los-equipos)
13. [Pruebas de convergencia y conectividad](#13-pruebas-de-convergencia-y-conectividad)
14. [Tablas de enrutamiento de los routers frontera](#14-tablas-de-enrutamiento-de-los-routers-frontera)
15. [Pruebas de conmutación por error](#15-pruebas-de-conmutación-por-error)
16. [Cumplimiento de los requerimientos](#16-cumplimiento-de-los-requerimientos)
17. [Problemas técnicos durante la construcción](#17-problemas-técnicos-durante-la-construcción)
18. [Cómo abrir y verificar el proyecto](#18-cómo-abrir-y-verificar-el-proyecto)
19. [Estructura de la entrega](#19-estructura-de-la-entrega)
20. [Referencias](#20-referencias)

---

## 1. Parámetros de identificación

El enunciado liga los parámetros de red al carné del coordinador y al número de grupo:

| Parámetro | Regla del enunciado | Valor aplicado |
|---|---|---|
| `XX` | Dos últimos dígitos del carné del coordinador (202300476) | **76** |
| `Y` | Último dígito del número de grupo (grupo 2) | **2** |
| Dominio VTP | `bantechXX` | **bantech76** |
| Contraseña VTP | fija | **cisco** |
| Bloque de direcciones | propuesto por el estudiante, ligado a `XX` | **10.76.0.0/16** |

Con `Y = 2`, los ID de VLAN quedan así. En el Data Center, `1Y+1` se interpreta como el número `1Y`
más uno: 12 + 1 = **13**.

| Sede | VLAN | Regla | ID |
|---|---|---|---:|
| Occidente | Cajas | `1Y` | **12** |
| Occidente | Asesores | `2Y` | **22** |
| Occidente | Gerencia | `3Y` | **32** |
| Occidente | Seguridad | `4Y` | **42** |
| Norte | Análisis | `5Y` | **52** |
| Norte | Auditoría | `6Y` | **62** |
| Norte | Legal | `7Y` | **72** |
| Oriente | Bóveda | `8Y` | **82** |
| Oriente | Plataforma | `9Y` | **92** |
| Data Center | Core_BD | `1Y+1` | **13** |
| Data Center | Web_Apps | `2Y+1` | **23** |
| Data Center | NOC | `3Y+1` | **33** |

---

## 2. Resumen de la solución

| Dominio | Protocolo | Red base | Inter-VLAN | Alta disponibilidad |
|---|---|---|---|---|
| Backbone nacional | OSPF área 0 | 10.76.0.0/24 (enlaces /30) | — | Triángulo de núcleo con 3 equipos de capa 3 y EtherChannel L3 LACP entre CORE-1 y CORE-2 |
| Sede Occidente | RIPv2 | 10.76.10.0/24 | Router-on-a-Stick en R-Occidente | VTP v2 (servidor y clientes) |
| Sede Norte | EIGRP AS 100 | 10.76.20.0/24 | Router-on-a-Stick en R-Norte | Triángulo de switches con Rapid PVST+ y Root Bridge forzado |
| Sede Oriente | OSPF área 0 | 10.76.30.0/24 | SVIs en MS1 y MS2 | HSRP v2 con preempt y track, acceso con doble conexión y EtherChannel LACP |
| Data Center y Sede Central | Rutas estáticas | 10.76.40.0/24 | SVIs en SW-DC-DIST | EtherChannel LACP de 2 cables hacia cada switch de acceso |

Hay **tres puntos de redistribución**, uno más que el mínimo de dos que pide el enunciado:

| Punto | Equipo | Dominios que une |
|---|---|---|
| 1 | **CORE-1** | RIPv2 (Occidente) ↔ OSPF |
| 2 | **CORE-3** | EIGRP 100 (Norte) ↔ OSPF |
| 3 | **R-Central1** | Rutas estáticas (Data Center) → OSPF |

**Totales:** 21 equipos de red, 84 dispositivos finales (7 por VLAN), 12 VLANs y 12 subredes punto a
punto /30.

---

## 3. Topología

### 3.1 Vista del lienzo completo

![Topología BanTech GT](docs/img/topologia.png)

### 3.2 Vista en la ventana de Packet Tracer

![Topología en Packet Tracer](docs/img/topologia-ventana-packet-tracer.png)

### 3.3 Identificación de áreas y protocolos en el simulador

Para cumplir con el requisito de *"identificar en la topología dentro del simulador el área donde se
aplicó cada protocolo de enrutamiento"*, el lienzo se organizó en zonas de color. Cada zona tiene un
encabezado con el nombre de la sede, el protocolo y la red base.

| Zona | Color | Encabezado en el lienzo |
|---|---|---|
| Sede Norte | Verde menta | `SEDE NORTE \| Centro de Autorizacion de Creditos \| EIGRP AS 100 \| 10.76.20.0/24` |
| Backbone | Lavanda | `BACKBONE NACIONAL \| Nucleo OSPF Area 0 \| 10.76.0.0/24 (/30)` |
| Data Center | Rosa | `DATA CENTER Y SEDE CENTRAL \| Rutas estaticas \| 10.76.40.0/24` |
| Sede Occidente | Durazno | `SEDE OCCIDENTE \| Agencia Regional Comercial \| RIPv2 \| 10.76.10.0/24` |
| Sede Oriente | Azul | `SEDE ORIENTE \| Centro Financiero Legado \| OSPF + HSRP \| 10.76.30.0/24` |
| Leyenda | Crema | Colores de zona, puntos de redistribución, medios físicos, alta disponibilidad, tabla de VLANs, VLSM e inter-VLAN por sede |

Sobre el lienzo también hay notas con:

- la función de cada equipo de red, con una ★ en los puntos de redistribución, el Root Bridge y el servidor VTP;
- la subred /30 de cada enlace del backbone y los EtherChannel;
- la VLAN, la red y el gateway de cada grupo de hosts.

---

## 4. Inventario de equipos

### 4.1 Equipos de red

| Equipo | Modelo | Módulos | Zona | Función |
|---|---|---|---|---|
| `CORE-1` | Catalyst 3560-24PS | — | Backbone | Núcleo OSPF; **redistribución RIPv2 ↔ OSPF** |
| `CORE-2` | Catalyst 3560-24PS | — | Backbone | Núcleo OSPF |
| `CORE-3` | Cisco 2911 | HWIC-1GE-SFP (0/0), HWIC-2T (0/1) | Backbone | Núcleo OSPF; **redistribución EIGRP ↔ OSPF**; extremo de la fibra y del serial |
| `R-Occidente` | Cisco 2911 | — | Occidente | Router de borde; Router-on-a-Stick; RIPv2 |
| `SW-OCC-DIST` | Catalyst 2960-24TT | — | Occidente | Switch principal; **servidor VTP** |
| `SW-OCC-ACC1` | Catalyst 2960-24TT | — | Occidente | Acceso (Cajas y Gerencia); cliente VTP |
| `SW-OCC-ACC2` | Catalyst 2960-24TT | — | Occidente | Acceso (Asesores y Seguridad); cliente VTP |
| `R-Norte` | Cisco 2911 | HWIC-2T (0/0) | Norte | Router de borde; Router-on-a-Stick; EIGRP 100 |
| `SW-NOR-DIST` | Catalyst 2960-24TT | — | Norte | Switch principal; **Root Bridge** (prioridad 4096) |
| `SW-NOR-ACC1` | Catalyst 2960-24TT | — | Norte | Acceso (Análisis y Legal); root secundario (8192) |
| `SW-NOR-ACC2` | Catalyst 2960-24TT | — | Norte | Acceso (Auditoría y Legal) |
| `MS1` | Catalyst 3560-24PS | — | Oriente | Multicapa; **HSRP activo**; Root Bridge VLAN 82 y 92 |
| `MS2` | Catalyst 3560-24PS | — | Oriente | Multicapa; HSRP en espera (standby) |
| `SW-ORI-ACC1` | Catalyst 2960-24TT | — | Oriente | Acceso (Bóveda), con conexión a MS1 y MS2 |
| `SW-ORI-ACC2` | Catalyst 2960-24TT | — | Oriente | Acceso (Plataforma), con conexión a MS1 y MS2 |
| `R-Central1` | Cisco 2911 | HWIC-1GE-SFP (0/0) | Data Center | Router de borde; **redistribución de estáticas a OSPF** |
| `R-Central2` | Cisco 2911 | — | Data Center | Router de acceso al segmento estático |
| `SW-DC-DIST` | Catalyst 3560-24PS | — | Data Center | Distribución de capa 3; SVIs de las VLANs 13, 23 y 33 |
| `SW-DC-ACC1` | Catalyst 2960-24TT | — | Data Center | Acceso a servidores Core_BD |
| `SW-DC-ACC2` | Catalyst 2960-24TT | — | Data Center | Acceso a servidores Web_Apps |
| `SW-DC-ACC3` | Catalyst 2960-24TT | — | Data Center | Acceso a estaciones del NOC |

### 4.2 Dispositivos finales

El enunciado pide un mínimo de 7 dispositivos finales por área. Se colocaron **7 por cada VLAN**, 84 en total:

| Sede | Dispositivos | Tipo | VLAN | Switch | Puertos |
|---|---|---|---:|---|---|
| Occidente | `PC-CAJ-01` … `07` | PC | 12 | SW-OCC-ACC1 | Fa0/1–7 |
| Occidente | `LAP-GER-01` … `07` | Laptop | 32 | SW-OCC-ACC1 | Fa0/8–14 |
| Occidente | `PC-ASE-01` … `07` | PC | 22 | SW-OCC-ACC2 | Fa0/1–7 |
| Occidente | `PC-SEG-01` … `07` | PC | 42 | SW-OCC-ACC2 | Fa0/8–14 |
| Norte | `PC-ANA-01` … `07` | PC | 52 | SW-NOR-ACC1 | Fa0/1–7 |
| Norte | `LAP-LEG-01` … `04` | Laptop | 72 | SW-NOR-ACC1 | Fa0/8–11 |
| Norte | `PC-AUD-01` … `07` | PC | 62 | SW-NOR-ACC2 | Fa0/1–7 |
| Norte | `LAP-LEG-05` … `07` | Laptop | 72 | SW-NOR-ACC2 | Fa0/8–10 |
| Oriente | `PC-BOV-01` … `07` | PC | 82 | SW-ORI-ACC1 | Fa0/1–7 |
| Oriente | `LAP-PLA-01` … `07` | Laptop | 92 | SW-ORI-ACC2 | Fa0/1–7 |
| Data Center | `SRV-BD-01` … `07` | Servidor | 13 | SW-DC-ACC1 | Fa0/1–7 |
| Data Center | `SRV-WEB-01` … `07` | Servidor | 23 | SW-DC-ACC2 | Fa0/1–7 |
| Data Center | `PC-NOC-01` … `07` | PC | 33 | SW-DC-ACC3 | Fa0/1–7 |

El área *Legal* (VLAN 72) tiene usuarios en los dos switches de acceso de la Sede Norte. Así se
demuestra que una misma VLAN atraviesa el anillo por los troncales.

---

## 5. Backbone nacional

### 5.1 Equipos e interconexión

El núcleo está formado por **tres equipos de capa 3**: `CORE-1`, `CORE-2` y `CORE-3`. Se conectan en
**triángulo**, así que cada uno tiene dos caminos hacia los otros dos. Si se cae cualquier enlace del
núcleo, OSPF recalcula por el lado opuesto del triángulo.

- **CORE-1 ↔ CORE-2:** canal principal de transporte, un **EtherChannel de capa 3 con LACP** (`Port-channel1`)
  formado por `Gi0/1` y `Gi0/2` en ambos extremos. Suma dos enlaces Gigabit bajo una sola IP
  (10.76.0.0/30) y sigue operando aunque se corte uno de los dos cables.
- **CORE-1 ↔ CORE-3** y **CORE-2 ↔ CORE-3:** enlaces Ethernet punto a punto.
- **CORE-3 ↔ R-Central1:** **fibra óptica** sobre módulos `HWIC-1GE-SFP`.
- **CORE-3 ↔ R-Norte:** **serial WAN**. CORE-3 es el extremo DCE (`clock rate 2000000`).

Los routers de borde se conectan así:

| Sede | Router de borde | Se conecta a | Protocolo en el enlace |
|---|---|---|---|
| Occidente | R-Occidente | CORE-1 | RIPv2 |
| Norte | R-Norte | CORE-3 (serial) | EIGRP 100 |
| Oriente | MS1 y MS2 | CORE-1 y CORE-2 (uno cada uno) | OSPF |
| Data Center | R-Central1 | CORE-2 (Ethernet) y CORE-3 (fibra) | OSPF |

R-Central1 y la pareja MS1/MS2 tienen **doble conexión** al núcleo. Si cae un equipo del núcleo, el
Data Center y la Sede Oriente siguen alcanzables por el otro.

### 5.2 Medios físicos del backbone

| Medio | Enlace | Puertos | Cable en Packet Tracer |
|---|---|---|---|
| **Fibra óptica** | CORE-3 ↔ R-Central1 | Gi0/0/0 ↔ Gi0/0/0 | Fiber |
| **Serial WAN** | CORE-3 ↔ R-Norte | Se0/1/0 (DCE) ↔ Se0/0/0 | Serial DCE |
| **EtherChannel L3 (Ethernet)** | CORE-1 ↔ CORE-2 | Gi0/1–2 ↔ Gi0/1–2 | Cobre cruzado ×2 |
| **Ethernet local** | CORE-1 ↔ CORE-3, CORE-2 ↔ CORE-3, núcleo ↔ sedes, R-Central1 ↔ R-Central2 | ver tabla 6.1 | Cobre directo o cruzado |

### 5.3 Dominios de enrutamiento y función de cada uno

| Dominio | Dónde opera | Función dentro de BanTech GT |
|---|---|---|
| **OSPF (proceso 1, área 0)** | CORE-1, CORE-2, CORE-3, R-Central1, MS1, MS2 | Protocolo del núcleo. Converge rápido por estado de enlace, elige rutas por costo y balancea entre caminos de igual costo (se ve en las tablas: varias rutas con dos *next-hop*). Todas las demás rutas se inyectan aquí. |
| **EIGRP (AS 100)** | CORE-3 (Se0/1/0) y R-Norte | Dominio de expansión regional. Corre sobre el enlace WAN serial hacia la Sede Norte. |
| **RIPv2** | CORE-1 (Fa0/2) y R-Occidente | Dominio de red legada. Una agencia pequeña con un solo camino hacia el núcleo. |
| **Rutas estáticas** | R-Central1, R-Central2 y SW-DC-DIST | Segmento de alta seguridad del Data Center. No intercambia rutas con nadie: se integra al backbone solo cuando R-Central1 redistribuye sus estáticas. |

Router-ID de OSPF:

| Equipo | Router-ID |
|---|---|
| CORE-1 | 1.1.1.1 |
| CORE-2 | 2.2.2.2 |
| CORE-3 | 3.3.3.3 |
| R-Central1 | 4.4.4.4 |
| MS1 | 5.5.5.5 |
| MS2 | 6.6.6.6 |

### 5.4 Asignación de protocolos a las sedes

El Data Center ya tenía asignadas las rutas estáticas. Las otras tres sedes se asignaron así:

- **Occidente → RIPv2.** La agencia comercial tiene un único router de borde con cuatro LAN detrás y un
  solo enlace al núcleo. No hay caminos alternos que elegir, así que la métrica por saltos de RIP es
  suficiente. RIPv2 es el protocolo más sencillo de administrar, que es justo lo que pide la necesidad
  operativa (*"fácilmente administrable"*). Además, envía la máscara en sus actualizaciones, lo que
  permite usar VLSM.
- **Norte → EIGRP.** La sede llega al núcleo por el enlace WAN serial, que representa la conectividad
  regional. Corresponde al *"dominio de expansión regional con EIGRP"*. EIGRP tiene en cuenta el ancho
  de banda y el retardo del enlace serial en su métrica y converge rápido gracias a DUAL. Eso conviene
  en una sede cuya caída detiene la aprobación de créditos a nivel nacional.
- **Oriente → OSPF.** El enunciado indica que *"el diseño del backbone ya incluyó dos switches multicapa
  (MS1 y MS2)"* para esta sede, así que MS1 y MS2 forman parte del núcleo y hablan su mismo protocolo.
  Cada uno sube a un equipo distinto del núcleo (MS1 a CORE-1 y MS2 a CORE-2). OSPF converge en
  segundos ante la caída de cualquiera de los dos. Con RIP o con redistribución adicional se perdería
  parte de la ventaja que da HSRP en esa sede.

### 5.5 Puntos de redistribución

**CORE-1: RIPv2 ↔ OSPF.** Las redes de Occidente entran a OSPF como externas tipo 2 (`O E2`). Hacia
Occidente, todo el backbone se anuncia por RIP con métrica semilla de 2 saltos.

```
router ospf 1
 router-id 1.1.1.1
 redistribute rip subnets
 passive-interface FastEthernet0/2
 network 10.76.0.0 0.0.0.3 area 0
 network 10.76.0.4 0.0.0.3 area 0
 network 10.76.0.12 0.0.0.3 area 0
 network 10.76.0.16 0.0.0.3 area 0
!
router rip
 version 2
 redistribute ospf 1 metric 2
 passive-interface default
 no passive-interface FastEthernet0/2
 network 10.0.0.0
 no auto-summary
```

**CORE-3: EIGRP ↔ OSPF.** Las redes de Norte entran a OSPF como `O E2`. Hacia Norte, el backbone se
anuncia como `D EX` con métrica semilla `1544 20000 255 1 1500`: ancho de banda de 1544 kbps, retardo
de 20000 decenas de µs, confiabilidad 255, carga 1 y MTU de 1500.

```
router eigrp 100
 redistribute ospf 1 metric 1544 20000 255 1 1500
 network 10.76.0.32 0.0.0.3
!
router ospf 1
 router-id 3.3.3.3
 redistribute eigrp 100 subnets
 passive-interface Serial0/1/0
 network 10.76.0.4 0.0.0.3 area 0
 network 10.76.0.8 0.0.0.3 area 0
 network 10.76.0.28 0.0.0.3 area 0
 network 10.76.0.32 0.0.0.3 area 0
```

**R-Central1: estáticas → OSPF.** R-Central1 conoce las VLANs del Data Center por rutas estáticas hacia
R-Central2 y las publica en OSPF. El resto de las sedes alcanza el Data Center sin que el segmento
estático participe en ningún protocolo dinámico. Es la *"integración controlada"* que pide el enunciado.

```
ip route 10.76.40.0  255.255.255.224 10.76.0.38
ip route 10.76.40.32 255.255.255.224 10.76.0.38
ip route 10.76.40.64 255.255.255.240 10.76.0.38
ip route 10.76.0.40  255.255.255.252 10.76.0.38
!
router ospf 1
 router-id 4.4.4.4
 redistribute static subnets
 passive-interface GigabitEthernet0/1
 network 10.76.0.24 0.0.0.3 area 0
 network 10.76.0.28 0.0.0.3 area 0
 network 10.76.0.36 0.0.0.3 area 0
```

La palabra clave `subnets` es necesaria en OSPF. Sin ella solo se redistribuirían redes con clase
completa y se perderían todas las subredes VLSM.

---
