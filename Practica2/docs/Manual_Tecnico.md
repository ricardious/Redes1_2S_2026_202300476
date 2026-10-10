# Manual técnico - Red de Ciudad Cayalá

Carnet: **202300476**. Simulador: Cisco Packet Tracer 9.0.1. Configuración de switches: CLI.

Archivo del proyecto: [`Practica2_202300476.pkt`](../packet-tracer/Practica2_202300476.pkt).

## 1. Alcance del modelo compacto

La entrega principal contiene **30 PCs instalados**, seis por zona, cinco switches principales 3560-24PS y cinco secundarios 2960-24TT. Los 60/28/12/50/7 hosts del enunciado son la demanda proyectada que dimensiona las subredes. No se afirma que los 157 hosts estén instalados en este archivo compacto.

Se eligió este formato compacto, siguiendo el estilo de los diagramas de ejemplo, porque permite leer la topología y demostrar todos los protocolos. Una primera versión con los 157 hosts instalados se usó como punto de partida ([captura](evidencias/capturas/01_etapa1_red_completa_157pcs.png)).

El objetivo SMART del enunciado V2 especifica conectividad intra-VLAN y aislamiento entre VLANs. En el foro del curso, el auxiliar aclaró: *"no hay que implementar nada de capa 3, todo será por medio de switches"*. La "conectividad entre zonas" se demuestra a nivel de enlace: trunks, VTP, STP y LACP entre los switches de todas las zonas, y pings dentro de cada VLAN que atraviesan varios switches.

Se configuran gateways reservados en los PCs, sin interfaces de gateway activas ni enrutamiento inter-VLAN. Un trunk transporta VLANs, pero no comunica unas VLANs con otras.

## 2. Investigación y análisis de las zonas

Los establecimientos se identificaron en el [directorio oficial de Cayalá](https://cayala.com/directorio/); Cayibel tiene además [su página oficial](https://www.cayala.com/cayibel). Los perfiles de tráfico y criticidad siguientes son supuestos de ingeniería para la práctica, no datos internos de los establecimientos.

| Zona | Rol y usuarios | Demanda | Tráfico estimado de diseño | Criticidad y decisión |
|---|---|---:|---|---|
| 1 - Área comercial | Personal de tiendas y operaciones comerciales | 60 hosts | Alto por cantidad; POS, inventario y aplicaciones web | Alta: dos caminos por el triángulo y LACP con Z4 |
| 2 - Cinépolis | Personal de boletería y operación del cine | 28 hosts | Medio; transacciones y sistemas operativos, sin asumir streaming de películas | Media: nodo de distribución del triángulo |
| 3 - REGUS | Personal y usuarios de oficinas | 12 hosts | Medio; documentos, reuniones y aplicaciones de oficina | Media: ramal en estrella, menor costo y complejidad |
| 4 - Cayibel | Personal de establecimientos gastronómicos | 50 hosts | Alto por concurrencia de pedidos y pagos | Alta: LACP y caminos alternativos |
| 5 - Art and Crafts | Personal y usuarios del establecimiento | 7 hosts | Bajo; operaciones administrativas y consultas web | Baja: ramal sencillo |

La demanda comercial y gastronómica justifica agregar capacidad entre sus switches principales. Las oficinas y el establecimiento de menor demanda no reciben el mismo número de conexiones: la topología responde a las diferencias de tráfico y disponibilidad.

## 3. Topología híbrida y redundancia

Se combinan **estrella** en el acceso y los ramales de Z3/Z5, con un **triángulo redundante** entre los principales Z1/Z2/Z4. El lado Z1-Z4 es un EtherChannel con dos enlaces Gigabit físicos. Los secundarios de Z1 y Z4 tienen, además, un trunk de respaldo hacia el principal de la otra zona crítica.

Cada zona contiene dos PCs en el principal (PC 01 en Fa0/10 y PC 04 en Fa0/11) y cuatro en el secundario (PC 02, 03, 05 y 06 en Fa0/2 a Fa0/5). Así, los pings entre un PC del principal y uno del secundario recorren un trunk.

![Topología final](evidencias/capturas/03_etapa3_topologia_final_30pcs.png)

El diagrama **no tiene cables cruzados**: se comprobó geométricamente que ningún cable corta a otro ni pasa sobre un dispositivo o una etiqueta. Z2 es el **núcleo central** de la fila superior, con los ramales Z3 y Z5 saliendo en horizontal a cada lado. En cada zona, los cuatro PCs del secundario forman un abanico alrededor de él y los dos del principal quedan a sus lados. Abajo, Z4 ocupa la izquierda y Z1 la derecha, unidas por **Po1 en horizontal**; el respaldo de SW-Z1-A1 hacia SW-Z4-P pasa por encima de Po1 y el de SW-Z4-A1 hacia SW-Z1-P por debajo. Cada zona tiene fondo pastel y borde de su color, con un rótulo (VLAN, subred, criticidad y rol VTP/STP del principal). Los enlaces entre switches muestran sus puertos; las VLANs permitidas y los rangos de IP están en los paneles inferiores. La leyenda explica colores, tipos de cable, estados de puerto y listas de trunk. Los fondos se dibujaron con líneas de 1 px dejando un halo de 6 px alrededor de cada cable, porque Packet Tracer dibuja las líneas por encima de los cables.

Los ramales de Z3/Z5 tienen un solo enlace ascendente y aceptan ese punto de fallo por el nivel de disponibilidad asumido. LACP protege frente a la pérdida de un miembro; STP protege frente a bucles y habilita caminos alternativos. Dos cables al mismo vecino no equivalen a redundancia de equipos.

### Inventario y crecimiento

Hay diez switches de 26 puertos físicos cada uno: 260 puertos. Se utilizan 56 extremos de puertos de switches (30 para PCs y 26 para los 13 enlaces entre switches). Los **204 puertos físicos libres** están en VLAN 999 y apagados.

El modelo compacto demuestra protocolos; no representa todos los puertos de acceso de la expansión final. Para instalar los 60 y 50 hosts de Z1/Z4 se requiere al menos un switch de acceso adicional en cada una y revisar su distribución de puertos. La primera versión con 157 PCs (ver su captura en las evidencias) usaba ese acceso ampliado.

## 4. VLANs y direccionamiento

Se utiliza VLSM dentro de 10.76.0.0/24. Se reserva el primer host utilizable como gateway y se asignan IPs desde el siguiente.

| Zona | VLAN / nombre | Subred | Máscara | Hosts proyectados | Gateway reservado |
|---|---|---|---|---:|---|
| Z1 | 16 / COMERCIO | 10.76.0.0/26 | 255.255.255.192 | 60 | 10.76.0.1 |
| Z2 | 26 / CINEPOLIS | 10.76.0.128/27 | 255.255.255.224 | 28 | 10.76.0.129 |
| Z3 | 36 / REGUS | 10.76.0.160/28 | 255.255.255.240 | 12 | 10.76.0.161 |
| Z4 | 46 / CAYIBEL | 10.76.0.64/26 | 255.255.255.192 | 50 | 10.76.0.65 |
| Z5 | 56 / ARTE | 10.76.0.176/28 | 255.255.255.240 | 7 | 10.76.0.177 |
| Administración de trunks | 99 / NATIVA_ADMIN | Sin subred configurada | No aplica | Sin hosts | No aplica |
| Puertos no utilizados | 999 / BLACKHOLE | Sin subred configurada | No aplica | Sin hosts activos | No aplica |

Las subredes /26, /27 y /28 admiten 62, 30 y 14 IPs útiles respectivamente. Aun reservando el gateway, alcanzan para la demanda definida. VLAN 99 se utiliza como nativa consistente; no se configuró una SVI de administración ni acceso remoto.

### Hosts instalados

| Host | VLAN | IP | Máscara | Gateway reservado | Conexión |
|---|---|---|---|---|---|
| PC-Z1-01 | 16 | 10.76.0.2 | 255.255.255.192 | 10.76.0.1 | SW-Z1-P Fa0/10 |
| PC-Z1-02 | 16 | 10.76.0.3 | 255.255.255.192 | 10.76.0.1 | SW-Z1-A1 Fa0/2 |
| PC-Z1-03 | 16 | 10.76.0.4 | 255.255.255.192 | 10.76.0.1 | SW-Z1-A1 Fa0/3 |
| PC-Z1-04 | 16 | 10.76.0.5 | 255.255.255.192 | 10.76.0.1 | SW-Z1-P Fa0/11 |
| PC-Z1-05 | 16 | 10.76.0.6 | 255.255.255.192 | 10.76.0.1 | SW-Z1-A1 Fa0/4 |
| PC-Z1-06 | 16 | 10.76.0.7 | 255.255.255.192 | 10.76.0.1 | SW-Z1-A1 Fa0/5 |
| PC-Z2-01 | 26 | 10.76.0.130 | 255.255.255.224 | 10.76.0.129 | SW-Z2-P Fa0/10 |
| PC-Z2-02 | 26 | 10.76.0.131 | 255.255.255.224 | 10.76.0.129 | SW-Z2-A1 Fa0/2 |
| PC-Z2-03 | 26 | 10.76.0.132 | 255.255.255.224 | 10.76.0.129 | SW-Z2-A1 Fa0/3 |
| PC-Z2-04 | 26 | 10.76.0.133 | 255.255.255.224 | 10.76.0.129 | SW-Z2-P Fa0/11 |
| PC-Z2-05 | 26 | 10.76.0.134 | 255.255.255.224 | 10.76.0.129 | SW-Z2-A1 Fa0/4 |
| PC-Z2-06 | 26 | 10.76.0.135 | 255.255.255.224 | 10.76.0.129 | SW-Z2-A1 Fa0/5 |
| PC-Z3-01 | 36 | 10.76.0.162 | 255.255.255.240 | 10.76.0.161 | SW-Z3-P Fa0/10 |
| PC-Z3-02 | 36 | 10.76.0.163 | 255.255.255.240 | 10.76.0.161 | SW-Z3-A1 Fa0/2 |
| PC-Z3-03 | 36 | 10.76.0.164 | 255.255.255.240 | 10.76.0.161 | SW-Z3-A1 Fa0/3 |
| PC-Z3-04 | 36 | 10.76.0.165 | 255.255.255.240 | 10.76.0.161 | SW-Z3-P Fa0/11 |
| PC-Z3-05 | 36 | 10.76.0.166 | 255.255.255.240 | 10.76.0.161 | SW-Z3-A1 Fa0/4 |
| PC-Z3-06 | 36 | 10.76.0.167 | 255.255.255.240 | 10.76.0.161 | SW-Z3-A1 Fa0/5 |
| PC-Z4-01 | 46 | 10.76.0.66 | 255.255.255.192 | 10.76.0.65 | SW-Z4-P Fa0/10 |
| PC-Z4-02 | 46 | 10.76.0.67 | 255.255.255.192 | 10.76.0.65 | SW-Z4-A1 Fa0/2 |
| PC-Z4-03 | 46 | 10.76.0.68 | 255.255.255.192 | 10.76.0.65 | SW-Z4-A1 Fa0/3 |
| PC-Z4-04 | 46 | 10.76.0.69 | 255.255.255.192 | 10.76.0.65 | SW-Z4-P Fa0/11 |
| PC-Z4-05 | 46 | 10.76.0.70 | 255.255.255.192 | 10.76.0.65 | SW-Z4-A1 Fa0/4 |
| PC-Z4-06 | 46 | 10.76.0.71 | 255.255.255.192 | 10.76.0.65 | SW-Z4-A1 Fa0/5 |
| PC-Z5-01 | 56 | 10.76.0.178 | 255.255.255.240 | 10.76.0.177 | SW-Z5-P Fa0/10 |
| PC-Z5-02 | 56 | 10.76.0.179 | 255.255.255.240 | 10.76.0.177 | SW-Z5-A1 Fa0/2 |
| PC-Z5-03 | 56 | 10.76.0.180 | 255.255.255.240 | 10.76.0.177 | SW-Z5-A1 Fa0/3 |
| PC-Z5-04 | 56 | 10.76.0.181 | 255.255.255.240 | 10.76.0.177 | SW-Z5-P Fa0/11 |
| PC-Z5-05 | 56 | 10.76.0.182 | 255.255.255.240 | 10.76.0.177 | SW-Z5-A1 Fa0/4 |
| PC-Z5-06 | 56 | 10.76.0.183 | 255.255.255.240 | 10.76.0.177 | SW-Z5-A1 Fa0/5 |

## 5. Cableado y trunks

Hosts a switches: cobre straight-through; switches entre sí: cobre crossover. Para una instalación física, el directo utiliza T568B en ambos extremos: blanco/naranja, naranja, blanco/verde, azul, blanco/azul, verde, blanco/café, café. El cruzado clásico combina T568A y T568B; no debe describirse como T568B en ambos extremos.

Se intentó fibra entre Z2 y Z3, pero el enlace no se creó en estos puertos del 3560. La ayuda local de Packet Tracer identifica al 3560 como equipo de configuración fija sin módulos adicionales. Se utilizó cobre Gigabit para conservar la conectividad. Los dos tipos de cable de cobre están presentes; esta versión no incluye un enlace óptico funcional.

| Extremo A | Extremo B | VLANs permitidas |
|---|---|---|
| SW-Z1-P Fa0/1 | SW-Z1-A1 Gi0/1 | 1,16,99 |
| SW-Z4-P Fa0/4 | SW-Z1-A1 Gi0/2 | 1,16,99 |
| SW-Z2-P Fa0/1 | SW-Z2-A1 Gi0/1 | 1,26,99 |
| SW-Z3-P Fa0/1 | SW-Z3-A1 Gi0/1 | 1,36,99 |
| SW-Z4-P Fa0/1 | SW-Z4-A1 Gi0/1 | 1,46,99 |
| SW-Z1-P Fa0/4 | SW-Z4-A1 Gi0/2 | 1,46,99 |
| SW-Z5-P Fa0/1 | SW-Z5-A1 Gi0/1 | 1,56,99 |
| SW-Z1-P Gi0/1 | SW-Z4-P Gi0/1 | 1,16,46,99 |
| SW-Z1-P Gi0/2 | SW-Z4-P Gi0/2 | 1,16,46,99 |
| SW-Z1-P Fa0/21 | SW-Z2-P Fa0/21 | 1,16,46,99 |
| SW-Z2-P Fa0/22 | SW-Z4-P Fa0/22 | 1,16,46,99 |
| SW-Z2-P Gi0/1 | SW-Z3-P Gi0/1 | 1,36,99 |
| SW-Z2-P Fa0/23 | SW-Z5-P Fa0/23 | 1,56,99 |

Todos los trunks tienen VLAN nativa 99 y listas explícitas. Criterio de poda: **una VLAN viaja por un trunk solo si detrás de ese enlace hay hosts de esa VLAN o un camino alternativo hacia ellos**. Los ramales Z3 y Z5 reciben únicamente su propia VLAN, la nativa y la de control; si recibieran las demás VLANs, los broadcasts de otras zonas los inundarían sin motivo. El anillo Z1-Z2-Z4 transporta solo **16 y 46**: los respaldos cruzados llevan VLAN 16 hasta SW-Z4-P y VLAN 46 hasta SW-Z1-P, y SW-Z2-P ofrece el camino alternativo si cae Po1. VLAN 26 no sale de Z2, y 36 y 56 cuelgan de SW-Z2-P por sus ramales. Como no hay capa 3, no hace falta llevar todas las VLANs a un punto central. En la simulación, el broadcast ARP de VLAN 36 termina en SW-Z2-P y no llega a Z1 ni a Z4. Con Po1 y los dos uplinks de acceso apagados, VLAN 16 y 46 siguen respondiendo a través de SW-Z2-P (4/4). VLAN 1 se permite para compatibilidad del tráfico de control PVST+ en esta versión de Packet Tracer; no tiene PCs asignados y no es la VLAN nativa. VLAN 999 no se permite en los trunks. Los parámetros de miembros y Port-channel deben coincidir.

Durante la verificación, al excluir VLAN 1, la simulación mostró BPDUs que los 3560 no entregaban a un proceso STP y volvían a difundir, con pérdidas de cola y bloqueos en puertos de PCs. Permitir explícitamente VLAN 1 en los trunks eliminó el fallo observado. Es un ajuste de compatibilidad comprobado en el simulador, no una afirmación de que todo switch real requiera transportar datos de VLAN 1. Se fijó SW-Z1-P como root de VLAN 1 con prioridad 4096 y SW-Z4-P como respaldo con 8192.

## 6. VTP

Dominio: `202300476`. Versión: 2 (`vtp version 2` en los servidores; los clientes la aprenden). Contraseña de laboratorio: `Cayala476Lab`.

Los cinco principales son servidores; los cinco secundarios son clientes. Comparten una única base de VLANs: no son cinco dominios independientes. Las VLANs se administran inicialmente desde SW-Z1-P y se verifica su aprendizaje en clientes mediante `show vtp status` y `show vlan brief`.

El número de revisión sube cada vez que Packet Tracer carga el archivo, aunque no se creen ni borren VLANs (en las capturas es 160). Lo importante es que los diez switches muestren **la misma** revisión, y así ocurre en cada carga.

En una de las aperturas los diez switches aparecieron en **VTP versión 1** (`VTP V2 Mode: Disabled`). Se aplicó de nuevo `vtp version 2` en los servidores y se guardó con `write memory`; desde entonces la versión 2 se mantiene. Se comprueba con `show vtp status` al abrir el archivo.

Debe coincidir el dominio, la versión, la contraseña, el inventario de VLANs y la revisión. Antes de conectar un switch ajeno a esta red se debe revisar su estado VTP y su revisión para evitar que una base antigua reemplace la vigente.

## 7. Rapid PVST+ y Root Bridge

Se habilita `spanning-tree mode rapid-pvst` en todos los switches. El principal de cada zona es root de su propia VLAN con prioridad base 4096.

| VLAN | Root principal | Prioridad base | Respaldo | Prioridad base |
|---|---|---:|---|---:|
| 16 | SW-Z1-P | 4096 | SW-Z4-P | 8192 |
| 26 | SW-Z2-P | 4096 | SW-Z1-P (sin efecto: VLAN 26 no sale de Z2) | 8192 |
| 36 | SW-Z3-P | 4096 | SW-Z2-P | 8192 |
| 46 | SW-Z4-P | 4096 | SW-Z1-P | 8192 |
| 56 | SW-Z5-P | 4096 | SW-Z2-P | 8192 |
| 99, 999 | SW-Z1-P | 4096 | SW-Z4-P | 8192 |

Los otros switches tienen prioridad base 32768 para esas VLANs. IOS muestra la suma de la prioridad base y el ID de VLAN por la extensión de sistema: por ejemplo, 4112 para VLAN 16. No es una prioridad mal configurada.

PortFast y BPDU Guard se aplican exclusivamente a puertos de PCs. Los trunks y el EtherChannel participan normalmente en STP. Un puerto alternativo en estado discarding es normal; evita el bucle y queda disponible para reconvergencia.

## 8. EtherChannel LACP

El carnet termina en 6, un número par: corresponde LACP. Se configura `channel-group 1 mode active` y `channel-protocol lacp` en Gi0/1 y Gi0/2 de SW-Z1-P y SW-Z4-P. El Port-channel es de capa 2 y utiliza las mismas VLANs y nativa que los miembros.

La capacidad agregada nominal es 2 Gbit/s por sentido, repartida entre flujos según el algoritmo del equipo. Un único flujo no obtiene automáticamente 2 Gbit/s. Con un miembro caído, el canal puede continuar con 1 Gbit/s.

La comprobación principal es `show etherchannel summary`: `Po1(SU)`, protocolo `LACP` y ambos miembros `(P)`. En esta imagen IOS, `show lacp neighbor` no está implementado y no se utiliza como evidencia de éxito.

## 9. Dominios de colisión y broadcast

El modelo compacto tiene **43 enlaces físicos Ethernet**: 30 PC-switch y 13 switch-switch. Con el criterio académico de un dominio de colisión por segmento Ethernet conmutado, se cuentan **43 dominios**. Los dos cables del EtherChannel cuentan por separado: STP los ve como un puerto lógico, pero siguen siendo dos segmentos físicos. Los switches delimitan los segmentos.

En enlaces full-duplex no se producen colisiones Ethernet; el conteo de dominios no significa que existan colisiones observadas.

Hay cinco dominios de broadcast de usuarios, uno por VLAN. VLANs 99 y 999 son dos dominios lógicos adicionales configurados, sin PCs activos. VLAN 1 también participa en los trunks para compatibilidad del plano de control, sin hosts de usuarios: son ocho VLANs considerando los cinco usuarios, 99, 999 y la predeterminada 1. Las VLANs heredadas 1002-1005 permanecen en la base y no se usan para esta práctica.

## 10. Seguridad de acceso a los switches

Se aplica a los diez switches. No altera el plano de datos.

| Medida | Comando | Motivo |
|---|---|---|
| Contraseña de modo privilegiado | `enable secret Cayala476Enable` | Se guarda con hash MD5 (tipo 5). `enable algorithm-type` no existe en estas imágenes 12.2. |
| Cifrado de contraseñas en claro | `service password-encryption` | La clave de consola aparece como tipo 7 |
| Consola protegida | `line console 0` → `password Cayala476Con`, `login`, `exec-timeout 15 0`, `logging synchronous` | Sin la clave nadie entra a modo usuario; la sesión inactiva se cierra |
| VTY cerradas | `line vty 0 15` → `transport input none`, `login` | No hay IP de gestión; se cierra el acceso remoto explícitamente |
| Aviso legal | `banner motd #Acceso restringido - Red Ciudad Cayala - 202300476#` | Advertencia de acceso autorizado |

Para la evaluación: al abrir el CLI de un switch, introducir `Cayala476Con` y, tras `enable`, `Cayala476Enable`. Son contraseñas de laboratorio; en una red real no se publicarían. Como mejoras posibles quedan port-security en los puertos de PC, root guard en los trunks de ramal y una SVI de gestión con SSH.

## 11. Verificación y reproducción

Los scripts [CLI por switch](scripts/) contienen la configuración completa. Se comprobó, línea por línea, que coinciden con el running-config real de los 10 switches del archivo `.pkt`. Para reproducir desde cero: colocar y cablear según las tablas de las secciones 4 y 5, configurar dominio/roles/trunks, crear las siete VLANs en SW-Z1-P, esperar sincronización VTP, aplicar prioridades y puertos de acceso, configurar el EtherChannel y los PCs, y guardar con `write memory`.

Consultar [el informe](Informe_de_Desarrollo.md) para los resultados reales y la simulación ARP/ICMP. Las salidas `show` y las 46 capturas de Packet Tracer están en [`evidencias/`](evidencias/README.md).

`show ip route` en el 3560 sin routing muestra información de host y que no hay gateway predeterminado del switch. No prueba conectividad inter-VLAN ni contradice que los PCs tengan direcciones de gateway reservadas.
