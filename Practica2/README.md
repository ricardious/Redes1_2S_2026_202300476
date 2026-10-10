# Práctica 2 — Red de Ciudad Cayalá

**VLANs, VTP, Rapid PVST+ y EtherChannel (Capas 1 y 2)**

Red de **cinco zonas de Ciudad Cayalá** en Cisco Packet Tracer: **30 PCs y 10 switches**, con una topología híbrida que combina estrella y un triángulo redundante. Cada zona cuenta con un switch principal, uno secundario y seis PCs: dos conectados al principal y cuatro al secundario.

| | |
|---|---|
| **Estudiante** | Alex Ricardo Castañeda Rodríguez |
| **Carné** | 202300476 |
| **Curso** | Redes de Computadoras 1 — Segundo Semestre 2026 |
| **Ponderación** | 7 pts |
| **Archivo de topología** | [`packet-tracer/Practica2_202300476.pkt`](packet-tracer/Practica2_202300476.pkt) |
| **Versión de Packet Tracer** | 9.0.1 |

![Topología de la red en Packet Tracer](docs/evidencias/capturas/03_etapa3_topologia_final_30pcs.png)

---

## Resumen técnico

| Elemento | Configuración |
|---|---|
| **Direccionamiento** | VLSM sobre `10.76.0.0/24` (/26, /26, /27, /28, /28) |
| **VLANs de zona** | 🟦 16 Comercio · 🟥 26 Cinépolis · 🟧 36 REGUS · 🟩 46 Cayibel · 🟪 56 Arte |
| **VLANs especiales** | 99 nativa · 999 Blackhole (204 puertos libres, apagados) |
| **VTP** | Versión 2, dominio `202300476`: 5 Server y 5 Client |
| **STP** | Rapid PVST+; el principal de cada zona es raíz de su VLAN (prioridad 4096) |
| **EtherChannel** | Po1 LACP entre SW-Z1-P y SW-Z4-P (2 × Gigabit) |
| **Seguridad de acceso** | `enable secret`, consola con contraseña, VTY cerradas y banner |

---

## Documentación

| Documento | Contenido |
|---|---|
| 📘 [**Manual técnico**](docs/Manual_Tecnico.md) | Análisis de zonas, topología, VLSM, trunks, VTP, STP, LACP y dominios |
| 📗 [**Informe de desarrollo**](docs/Informe_de_Desarrollo.md) | Construcción paso a paso, pruebas con capturas y problemas resueltos |
| ⌨️ [**Scripts CLI**](docs/scripts/) | Configuración completa de los 10 switches |
| 🖼️ [**Evidencias**](docs/evidencias/README.md) | 46 capturas de Packet Tracer y salidas `show` de cada switch |
| 📋 [**Tabla de hosts**](docs/Manual_Tecnico.md#hosts-instalados) | IP, máscara, gateway y puerto de los 30 PCs |

---

## Alcance

El proyecto utiliza seis PCs por zona para las pruebas. El direccionamiento VLSM se dimensionó para las cantidades de hosts del enunciado: 60, 28, 12, 50 y 7, respectivamente.

---

## Revisión rápida

1. Abrir el archivo `.pkt` en Packet Tracer y esperar a que la red converja. Se puede utilizar **Fast Forward Time** para acelerar la espera.
2. Acceder a la consola de un switch con `Cayala476Con` y al modo privilegiado con `enable` y la contraseña `Cayala476Enable`.
3. Ejecutar los comandos de verificación:

| Comando | Dónde | Resultado esperado |
|---|---|---|
| `show vlan brief` | Cualquier switch | VLANs 16, 26, 36, 46, 56, 99 y 999 |
| `show vtp status` | Cualquier switch | Dominio `202300476`, versión 2, misma revisión |
| `show interfaces trunk` | Cualquier switch | Nativa 99 y listas explícitas |
| `show spanning-tree vlan 16` | SW-Z1-P | `This bridge is the root` |
| `show etherchannel summary` | SW-Z1-P o SW-Z4-P | `Po1(SU)`, `LACP` y dos miembros `(P)` |

Las pruebas de conectividad y redundancia se describen en el [informe de desarrollo](docs/Informe_de_Desarrollo.md#6-pruebas).

---

## Estructura

```text
Practica2/
├── README.md
├── packet-tracer/
│   └── Practica2_202300476.pkt
└── docs/
    ├── Manual_Tecnico.md
    ├── Informe_de_Desarrollo.md
    ├── scripts/              # configuración CLI de los 10 switches
    ├── evidencias/
    │   ├── capturas/         # 46 capturas de Packet Tracer
    │   └── salidas_switches/ # show y running-config por switch
    ├── enunciado/
    └── 0970_Practica_2_HC_2S2026.pdf
```
