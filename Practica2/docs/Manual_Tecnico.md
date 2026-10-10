# Manual técnico - Red de Ciudad Cayalá

Carnet: **202300476**. Simulador: Cisco Packet Tracer 9.0.1. Los switches se configuraron mediante la interfaz de línea de comandos (CLI).

Archivo del proyecto: [`Practica2_202300476.pkt`](../packet-tracer/Practica2_202300476.pkt).

## 1. Alcance

El proyecto contiene **30 PCs**, seis por zona, cinco switches principales 3560-24PS y cinco secundarios 2960-24TT. Las subredes se dimensionaron para los 60, 28, 12, 50 y 7 hosts indicados en el enunciado, aunque el archivo final utiliza seis PCs por zona para las pruebas.

El diseño partió de una versión con 157 hosts, documentada en esta [captura inicial](evidencias/capturas/01_etapa1_red_completa_157pcs.png). Su evolución hasta la versión final se describe en el [informe de desarrollo](Informe_de_Desarrollo.md#2-evolución-del-diseño).

Las pruebas verifican la conectividad dentro de cada VLAN, el aislamiento entre VLANs y el funcionamiento de los trunks, VTP, STP y LACP entre switches.

## 2. Investigación y análisis de las zonas

Los establecimientos se identificaron en el [directorio oficial de Cayalá](https://cayala.com/directorio/); Cayibel tiene además [su página oficial](https://www.cayala.com/cayibel). Los perfiles de tráfico y criticidad siguientes son supuestos de ingeniería para la práctica, no datos internos de los establecimientos.

| Zona | Rol y usuarios | Demanda | Tráfico estimado de diseño | Criticidad y decisión |
|---|---|---:|---|---|
| 1 - Área comercial | Personal de tiendas y operaciones comerciales | 60 hosts | Alto por cantidad; POS, inventario y aplicaciones web | Alta: dos caminos por el triángulo y LACP con Z4 |
| 2 - Cinépolis | Personal de boletería y operación del cine | 28 hosts | Medio; transacciones y sistemas operativos, sin asumir streaming de películas | Media: nodo de distribución del triángulo |
| 3 - REGUS | Personal y usuarios de oficinas | 12 hosts | Medio; documentos, reuniones y aplicaciones de oficina | Media: ramal en estrella, menor costo y complejidad |
| 4 - Cayibel | Personal de establecimientos gastronómicos | 50 hosts | Alto por concurrencia de pedidos y pagos | Alta: LACP y caminos alternativos |
| 5 - Art and Crafts | Personal y usuarios del establecimiento | 7 hosts | Bajo; operaciones administrativas y consultas web | Baja: ramal sencillo |

La demanda de las zonas comercial y gastronómica justifica una mayor capacidad entre sus switches principales. En las demás zonas se utilizan menos enlaces, de acuerdo con el tráfico y la disponibilidad estimados.

## 3. Topología híbrida y redundancia

La topología combina conexiones en estrella en el acceso y en los ramales de Z3 y Z5 con un triángulo redundante entre los switches principales de Z1, Z2 y Z4. La conexión Z1-Z4 utiliza un EtherChannel de dos enlaces Gigabit. Los switches secundarios de Z1 y Z4 disponen, además, de un trunk de respaldo hacia el principal de la otra zona crítica.

Cada zona contiene dos PCs en el principal (PC 01 en Fa0/10 y PC 04 en Fa0/11) y cuatro en el secundario (PC 02, 03, 05 y 06 en Fa0/2 a Fa0/5). Así, los pings entre un PC del principal y uno del secundario recorren un trunk.

![Topología final](evidencias/capturas/03_etapa3_topologia_final_30pcs.png)

Z2 conecta los ramales de Z3 y Z5. Z1 y Z4 están unidas por Po1 y cuentan con enlaces de respaldo entre sus switches principales y secundarios. Las zonas se identifican por color y rótulos con VLAN, subred, criticidad y rol VTP/STP. Los enlaces entre switches muestran sus puertos y los paneles inferiores resumen las VLANs permitidas y los rangos de IP.

Los ramales de Z3 y Z5 tienen un único enlace ascendente, por lo que dependen de esa conexión. LACP permite mantener el canal ante la pérdida de uno de sus miembros; STP evita bucles y habilita caminos alternativos. La redundancia de enlaces no sustituye a la redundancia de equipos.

### Inventario y crecimiento

Los diez switches suman 260 puertos físicos, 26 por equipo. Se utilizan 56: 30 para conectar los PCs y 26 para los 13 enlaces entre switches. Los **204 puertos restantes** están asignados a la VLAN 999 y deshabilitados.

Para instalar los 60 hosts de Z1 y los 50 de Z4 se requiere, como mínimo, un switch de acceso adicional en cada zona y una revisión de la distribución de puertos. La versión inicial de 157 PCs incluía esa ampliación.

## 4. VLANs y direccionamiento

El direccionamiento utiliza VLSM dentro de la red 10.76.0.0/24. La primera dirección utilizable de cada subred se reserva como gateway; las direcciones de los PCs se asignan a partir de la siguiente.

| Zona | VLAN / nombre | Subred | Máscara | Hosts proyectados | Gateway reservado |
|---|---|---|---|---:|---|
| Z1 | 16 / COMERCIO | 10.76.0.0/26 | 255.255.255.192 | 60 | 10.76.0.1 |
| Z2 | 26 / CINEPOLIS | 10.76.0.128/27 | 255.255.255.224 | 28 | 10.76.0.129 |
| Z3 | 36 / REGUS | 10.76.0.160/28 | 255.255.255.240 | 12 | 10.76.0.161 |
| Z4 | 46 / CAYIBEL | 10.76.0.64/26 | 255.255.255.192 | 50 | 10.76.0.65 |
| Z5 | 56 / ARTE | 10.76.0.176/28 | 255.255.255.240 | 7 | 10.76.0.177 |
| Administración de trunks | 99 / NATIVA_ADMIN | Sin subred configurada | No aplica | Sin hosts | No aplica |
| Puertos no utilizados | 999 / BLACKHOLE | Sin subred configurada | No aplica | Sin hosts activos | No aplica |

Las subredes /26, /27 y /28 admiten 62, 30 y 14 direcciones útiles, respectivamente. Su capacidad cubre la demanda definida, incluida la dirección reservada como gateway. La VLAN 99 se utiliza como nativa en todos los trunks.

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

Los PCs se conectan a los switches mediante cable de cobre directo (*straight-through*); las conexiones entre switches utilizan cable cruzado (*crossover*). Para una instalación física, el cable directo utiliza T568B en ambos extremos: blanco/naranja, naranja, blanco/verde, azul, blanco/azul, verde, blanco/café y café. El cable cruzado clásico combina T568A en un extremo y T568B en el otro.

Durante la construcción se intentó conectar Z2 y Z3 mediante fibra, pero los puertos disponibles del 3560 en Packet Tracer no permitieron establecer el enlace. La ayuda local lo identifica como un equipo de configuración fija, sin módulos adicionales. Se utilizó cobre Gigabit como alternativa; el proyecto final no incluye enlaces de fibra óptica.

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

Todos los trunks utilizan la VLAN nativa 99 y una lista explícita de VLANs permitidas. La selección considera los hosts accesibles por cada enlace y sus caminos de respaldo:

- Los ramales de Z3 y Z5 transportan la VLAN de su zona, la 99 y la 1.
- El triángulo Z1-Z2-Z4 transporta las VLANs de usuarios 16 y 46, además de la 99 y la 1. Los enlaces de respaldo extienden la VLAN 16 hasta SW-Z4-P y la 46 hasta SW-Z1-P; SW-Z2-P proporciona un camino alternativo cuando Po1 no está disponible.
- La VLAN 26 permanece en Z2. Las VLANs 36 y 56 llegan a SW-Z2-P por sus respectivos ramales, pero no se propagan hacia Z1 ni Z4.
- La VLAN 1 se permite por compatibilidad con el tráfico de control de PVST+ en esta versión de Packet Tracer. No tiene PCs asignados ni se utiliza como nativa. La VLAN 999 se excluye de todos los trunks.

La simulación ARP de la VLAN 36 y las pruebas de fallo verifican esta distribución. Al deshabilitar Po1 y los dos enlaces ascendentes de acceso de Z1 y Z4, los pings de las VLANs 16 y 46 mantuvieron cuatro respuestas de cuatro a través de SW-Z2-P. Los parámetros de trunk del Port-channel y de sus miembros deben coincidir.

Al excluir la VLAN 1 durante las pruebas, se observaron reenvíos de BPDUs, pérdidas de cola y bloqueos en puertos de PCs. Permitirla explícitamente en los trunks resolvió el problema en el simulador. Este ajuste corresponde al comportamiento observado en Packet Tracer. Para la VLAN 1 se estableció SW-Z1-P como puente raíz, con prioridad 4096, y SW-Z4-P como respaldo, con 8192.

## 6. VTP

Dominio: `202300476`. Versión: 2 (`vtp version 2` en los servidores; los clientes la aprenden). Contraseña de laboratorio: `Cayala476Lab`.

Los cinco switches principales actúan como servidores y los cinco secundarios como clientes dentro de un mismo dominio VTP. Las VLANs se crean inicialmente en SW-Z1-P y su propagación a los clientes se verifica con `show vtp status` y `show vlan brief`.

Durante las verificaciones se observó que el número de revisión aumentaba al abrir el archivo, sin crear ni eliminar VLANs. Las capturas muestran la revisión 160. Al revisar el proyecto, se debe comprobar que los diez switches compartan la misma revisión.

En una de las aperturas, los diez switches aparecieron en VTP versión 1 (`VTP V2 Mode: Disabled`). Se volvió a aplicar `vtp version 2` en los servidores y se guardó la configuración con `write memory`. Las verificaciones posteriores confirmaron la versión 2 mediante `show vtp status`.

Deben coincidir el dominio, la versión, la contraseña, el inventario de VLANs y la revisión. Antes de incorporar otro switch, se debe revisar su estado VTP para evitar que una base de VLANs no deseada, con una revisión superior, sustituya la vigente.

## 7. Rapid PVST+ y puente raíz

Todos los switches utilizan `spanning-tree mode rapid-pvst`. El switch principal de cada zona es el puente raíz (*root bridge*) de su VLAN, con prioridad base 4096.

| VLAN | Root principal | Prioridad base | Respaldo | Prioridad base |
|---|---|---:|---|---:|
| 16 | SW-Z1-P | 4096 | SW-Z4-P | 8192 |
| 26 | SW-Z2-P | 4096 | SW-Z1-P (sin efecto: VLAN 26 no sale de Z2) | 8192 |
| 36 | SW-Z3-P | 4096 | SW-Z2-P | 8192 |
| 46 | SW-Z4-P | 4096 | SW-Z1-P | 8192 |
| 56 | SW-Z5-P | 4096 | SW-Z2-P | 8192 |
| 99, 999 | SW-Z1-P | 4096 | SW-Z4-P | 8192 |

Los demás switches tienen prioridad base 32768 para esas VLANs. Por la extensión del identificador de sistema, IOS muestra la suma de la prioridad base y el ID de la VLAN; por ejemplo, 4112 para la VLAN 16.

PortFast y BPDU Guard se aplican exclusivamente a los puertos de PCs. Los trunks y el EtherChannel participan en STP. Los puertos alternativos permanecen en estado de descarte (*discarding*) para evitar bucles y quedan disponibles ante un cambio de topología.

## 8. EtherChannel LACP

Se utiliza LACP porque el último dígito del carnet es par. Las interfaces Gi0/1 y Gi0/2 de SW-Z1-P y SW-Z4-P se configuran con `channel-group 1 mode active` y `channel-protocol lacp`. El Port-channel y sus miembros comparten la VLAN nativa y la lista de VLANs permitidas.

La capacidad agregada nominal es 2 Gbit/s por sentido, repartida entre flujos según el algoritmo del equipo. Un único flujo no obtiene automáticamente 2 Gbit/s. Con un miembro caído, el canal puede continuar con 1 Gbit/s.

El estado del canal se verifica con `show etherchannel summary`: debe aparecer `Po1(SU)`, el protocolo `LACP` y ambos miembros en estado `(P)`. La imagen IOS utilizada no admite `show lacp neighbor`.

## 9. Dominios de colisión y broadcast

El modelo compacto tiene **43 enlaces físicos Ethernet**: 30 PC-switch y 13 switch-switch. Con el criterio académico de un dominio de colisión por segmento Ethernet conmutado, se cuentan **43 dominios**. Los dos cables del EtherChannel cuentan por separado: STP los ve como un puerto lógico, pero siguen siendo dos segmentos físicos. Los switches delimitan los segmentos.

En enlaces full-duplex no se producen colisiones Ethernet; el conteo de dominios no significa que existan colisiones observadas.

Hay cinco dominios de broadcast de usuarios, uno por cada VLAN de zona. Las VLANs 99 y 999 constituyen dos dominios lógicos adicionales, sin PCs activos. La VLAN 1 participa en los trunks por compatibilidad con el plano de control y tampoco tiene PCs asignados. En conjunto se consideran ocho VLANs: las cinco de usuarios, la 99, la 999 y la predeterminada 1. Las VLANs heredadas 1002-1005 permanecen en la base, pero no se utilizan en la práctica.

## 10. Seguridad de acceso a los switches

Los diez switches incorporan las siguientes medidas de protección de acceso:

| Medida | Comando | Motivo |
|---|---|---|
| Contraseña de modo privilegiado | `enable secret Cayala476Enable` | Se guarda con hash MD5 (tipo 5). `enable algorithm-type` no existe en estas imágenes 12.2. |
| Cifrado de contraseñas en claro | `service password-encryption` | La clave de consola aparece como tipo 7 |
| Consola protegida | `line console 0` → `password Cayala476Con`, `login`, `exec-timeout 15 0`, `logging synchronous` | Solicita contraseña y cierra la sesión tras 15 minutos de inactividad |
| VTY cerradas | `line vty 0 15` → `transport input none`, `login` | No hay IP de gestión; se cierra el acceso remoto explícitamente |
| Aviso legal | `banner motd #Acceso restringido - Red Ciudad Cayala - 202300476#` | Advertencia de acceso autorizado |

Para la evaluación: al abrir el CLI de un switch, introducir `Cayala476Con` y, tras `enable`, `Cayala476Enable`. Son contraseñas de laboratorio; en una red real no se publicarían. Como mejoras posibles quedan port-security en los puertos de PC, root guard en los trunks de ramal y una SVI de gestión con SSH.

## 11. Verificación y reproducción

Los [scripts CLI por switch](scripts/) contienen la configuración completa y se contrastaron con el `running-config` de los diez switches del archivo `.pkt`. Para reconstruir el proyecto:

1. Colocar los dispositivos y conectarlos según las tablas de las secciones 4 y 5.
2. Configurar el dominio VTP, los roles de los switches y los trunks.
3. Crear las siete VLANs en SW-Z1-P y esperar la sincronización VTP.
4. Aplicar las prioridades STP y configurar los puertos de acceso y el EtherChannel.
5. Asignar a los PCs las direcciones de la tabla de hosts.
6. Guardar la configuración de cada switch con `write memory`.

El [informe de desarrollo](Informe_de_Desarrollo.md) presenta los resultados de las pruebas y la simulación ARP/ICMP. Las salidas `show` y las 46 capturas de Packet Tracer se encuentran en el [índice de evidencias](evidencias/README.md).

La salida de `show ip route` del 3560 se incluye en el informe y muestra el estado del gateway predeterminado y la caché de redirecciones ICMP.
