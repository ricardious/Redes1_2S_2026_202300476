# Práctica 2 - Red de Ciudad Cayalá

Redes de Computadoras 1 · Segundo semestre 2026 · Carnet **202300476**.

La red consta de **30 PCs y 10 switches** distribuidos en cinco zonas. Cada zona cuenta con un switch principal, uno secundario y seis PCs: dos conectados al principal y cuatro al secundario. La configuración incluye las VLANs 16, 26, 36, 46 y 56, la VLAN nativa 99, la VLAN Blackhole 999, VTP v2, Rapid PVST+ y EtherChannel con LACP.

![Topología de la red en Packet Tracer](docs/evidencias/capturas/03_etapa3_topologia_final_30pcs.png)

## Archivos

| Entregable | Enlace |
|---|---|
| Proyecto de Packet Tracer | [Practica2_202300476.pkt](packet-tracer/Practica2_202300476.pkt) |
| Manual técnico | [Diseño y configuración](docs/Manual_Tecnico.md) |
| Informe de desarrollo | [Construcción, pruebas y resultados](docs/Informe_de_Desarrollo.md) |
| Configuración completa por switch | [Scripts CLI](docs/scripts/) |
| IPs y conexiones de los 30 PCs | [Manual técnico, tabla de hosts](docs/Manual_Tecnico.md#hosts-instalados) |
| Capturas de Packet Tracer (46) y salidas `show` | [Índice de evidencias](docs/evidencias/README.md) |

## Alcance

El proyecto utiliza seis PCs por zona para las pruebas. El direccionamiento VLSM se dimensionó para las cantidades de hosts del enunciado: 60, 28, 12, 50 y 7, respectivamente.

## Revisión rápida

1. Abrir el archivo `.pkt` en Packet Tracer y esperar a que la red converja. Se puede utilizar **Fast Forward Time** para acelerar la espera.
2. Acceder a la consola de un switch con `Cayala476Con` y al modo privilegiado con `enable` y la contraseña `Cayala476Enable`.
3. Revisar las VLANs, VTP y los trunks con `show vlan brief`, `show vtp status` y `show interfaces trunk`.
4. Comprobar STP con `show spanning-tree vlan 16` en SW-Z1-P y LACP con `show etherchannel summary` en SW-Z1-P o SW-Z4-P. El canal debe mostrar `Po1(SU)`, el protocolo `LACP` y dos miembros en estado `(P)`.

Las pruebas de conectividad y redundancia se describen en el [informe de desarrollo](docs/Informe_de_Desarrollo.md#6-pruebas).
