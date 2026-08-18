# Manual Técnico — Diseño físico de red

**Práctica 1 · QuetzalDev S.A.**
Redes de Computadoras 1 · Segundo Semestre 2026
Universidad de San Carlos de Guatemala · Facultad de Ingeniería

| | |
|---|---|
| **Estudiante** | Alex Ricardo Castañeda Rodríguez |
| **Carnet** | 202300476 |
| **Plano base** | [`docs/planos/Plano4.png`](planos/Plano4.png) — edificio de un nivel, 22 × 22 m |
| **Diagrama** | [`packet-tracer/quetzaldev-plano-fisico-202300476.png`](../packet-tracer/quetzaldev-plano-fisico-202300476.png) |
| **Archivo editable** | [`packet-tracer/QuetzalDev_Practica1_Diseno_Fisico.pkt`](../packet-tracer/QuetzalDev_Practica1_Diseno_Fisico.pkt) |

---

## 1. Resumen del diseño

Diseñé la infraestructura de Capa 1 para el edificio de QuetzalDev S.A. sobre el plano
arquitectónico asignado. El alcance es **exclusivamente físico**: distribución de equipos,
puntos de red, medios de transmisión, rutas de canalización y etiquetado. No configuré
direccionamiento IP, VLANs ni protocolo alguno, porque eso corresponde a prácticas
posteriores.

| Concepto | Valor |
|---|---|
| Superficie del edificio | 22 × 22 m (484 m²), un solo nivel |
| Departamentos con red | 8 |
| Puntos de red | 48 |
| Distribución de hosts | 30 PC de escritorio + 12 laptops + 6 servidores |
| Tomas de red | 23 (21 dobles + 2 triples) |
| Switches de acceso | 8 (uno por departamento) |
| Switch principal | 1, en el MDF |
| Enlaces totales | 56 (48 horizontales + 8 troncales) |
| Cable horizontal | 587.0 m de UTP Cat6 |
| Cable troncal | 147.2 m de UTP Cat6a |
| Canalización | 50 m de escalerilla metálica abierta |
| Topología física | Estrella por departamento; árbol (estrella extendida) a nivel de edificio |

---

## 2. Interpretación del plano arquitectónico

El plano base define once espacios. Los tomé tal cual, sin modificar muros ni dimensiones,
y sobre ellos ubiqué el equipo de red. Estas son las dimensiones que leí del plano y que
usé después para todos los cálculos de distancia:

| Espacio | Dimensiones | Área | ¿Lleva puntos de red? |
|---|---|---|---|
| Departamento de Recepción | 12 × 5 m | 60 m² | Sí — 4 |
| Departamento Legal | 10 × 5 m | 50 m² | Sí — 4 |
| Hall Central | 22 × 2 m | 44 m² | No — es ruta de canalización |
| Departamento de Recursos Humanos | 7 × 6 m | 42 m² | Sí — 8 |
| Baño | 2 × 2 m | 4 m² | No |
| Sala de Capacitación | 7 × 6 m | 42 m² | Sí — 10 |
| Dirección General | 8 × 6 m | 48 m² | Sí — 4 |
| Pasillo Secundario | 22 × 2 m | 44 m² | No — es ruta de canalización y aloja el MDF |
| Departamento de Diseño e Innovación | 7 × 7 m | 49 m² | Sí — 8 |
| Vestíbulo de Ingreso | 4 × 7 m | 28 m² | No |
| Departamento de Backend | 7 × 7 m | 49 m² | Sí — 7 |
| Data Center | 4 × 7 m | 28 m² | Sí — 3 |

Para trabajar con precisión, tracé el plano en Packet Tracer a una escala fija de
**100 unidades de lienzo = 1.00 m**, con la esquina exterior del edificio en el origen
(1.00, 1.00) m y la opuesta en (23.00, 23.00) m. Todas las distancias de este manual salen
de esas coordenadas, no de mediciones a ojo.

Dos observaciones del plano que condicionaron el diseño:

- Los dos corredores (Hall Central y Pasillo Secundario) **atraviesan el edificio de lado a
  lado** y están a 2 m uno del otro en el eje vertical. Eso los vuelve el lugar natural para
  la canalización troncal: desde ellos se alcanza cualquier departamento sin cruzar un
  espacio de trabajo.
- El plano **no incluye un cuarto de telecomunicaciones**. Ubicarlo era decisión mía y lo
  justifico en la sección 4.

---

## 3. Inventario de equipos

### 3.1 Equipo activo

| Cantidad | Equipo | Ubicación | Función |
|---:|---|---|---|
| 1 | Switch principal de 48 puertos | MDF (Pasillo Secundario) | Concentra los 8 enlaces troncales; raíz del árbol |
| 1 | Switch de acceso 24 p | Recepción | 4 puntos de red |
| 1 | Switch de acceso 24 p | Legal | 4 puntos de red |
| 1 | Switch de acceso 24 p | Recursos Humanos | 8 puntos de red |
| 1 | Switch de acceso 24 p | Sala de Capacitación | 10 puntos de red |
| 1 | Switch de acceso 24 p | Dirección General | 4 puntos de red |
| 1 | Switch de acceso 24 p | Diseño e Innovación | 8 puntos de red |
| 1 | Switch de acceso 24 p | Backend | 7 puntos de red |
| 1 | Switch de acceso 24 p | Data Center | 3 puntos de red |

### 3.2 Equipo pasivo y de soporte

| Cantidad | Elemento | Detalle |
|---:|---|---|
| 1 | Gabinete de pared 12U | MDF, con puerta con cerradura |
| 1 | Patch panel Cat6 de 48 puertos | MDF |
| 8 | Patch panel Cat6 de 12 puertos | Uno por departamento, junto a su switch |
| 2 | Organizador horizontal de cables 1U | MDF |
| 1 | UPS 1500 VA / 900 W | MDF |
| 23 | Toma de red sobre muro | 21 dobles + 2 triples |
| 48 | Jack keystone Cat6 | Uno por punto de red |
| 50 m | Escalerilla metálica abierta 200 × 60 mm | Ramales EM-01, EM-02 y EM-03 |
| 70 m | Canaleta 40 × 25 mm | Bajadas de la escalerilla a cada toma |
| 3 | Bobina de cable de 305 m | 2 de Cat6 y 1 de Cat6a |
| 104 | Patch cords | 48 de 3 m, 48 de 1 m y 8 de 1 m Cat6a |

---

## 4. Ubicación del cuarto de telecomunicaciones (MDF)

**Ubicación elegida:** sobre el **Pasillo Secundario**, en las coordenadas (13.50, 15.20) m,
como gabinete de pared adosado al muro norte del corredor.

### 4.1 Por qué ahí

**Centralidad medida, no supuesta.** El centro geométrico de la planta está en (12.00, 12.00) m.
El MDF quedó a 3.5 m de ese punto. Lo importante no es la cercanía al centro en sí, sino su
consecuencia: **el enlace troncal más largo del edificio mide 22.7 m de recorrido** (hasta
`SW-RECEPCION`, la esquina superior izquierda) y el promedio de los ocho troncales es de
12.5 m. Con esas cifras, todo enlace horizontal queda muy por debajo del límite de 90 m que
fija TIA/EIA-568 para el enlace permanente: **el más largo del diseño mide 15.0 m**, es decir
un 17 % del máximo permitido.

**Acceso a las dos rutas de canalización.** El Pasillo Secundario ya es uno de los dos
corredores que cruzan el edificio. Colocar el MDF sobre él significa que la escalerilla
troncal nace literalmente en el gabinete: no hace falta un tramo de canalización dedicado
para sacar los cables del cuarto.

**No consume área productiva.** Un gabinete de pared de 12U ocupa aproximadamente
0.60 × 0.45 m sobre el muro del corredor. Si hubiera puesto un cuarto de telecomunicaciones
convencional habría tenido que quitarle 4 m² a algún departamento, y el plano no tiene
espacio ocioso: los once espacios están asignados.

**Seguridad razonable.** El corredor es zona común y de tránsito, lo que a primera vista es
una desventaja. La mitigo con un gabinete cerrado con llave, montado a 1.80 m del piso, fuera
del alcance casual. Para 48 puntos de red y un solo nivel, el riesgo residual es aceptable.

### 4.2 Alternativas que descarté

| Alternativa | Por qué la descarté |
|---|---|
| Dentro del Data Center | Está en la esquina inferior derecha. El troncal a Recepción subiría de 22.7 m a **30.5 m**, y el promedio de los ocho troncales pasaría de 12.5 m a **19.1 m**: un 53 % más de recorrido. |
| Dentro del Vestíbulo de Ingreso | Es la zona de acceso público del edificio. Colocar ahí el equipo activo es la peor opción de seguridad física. |
| Cuarto dedicado quitando área a Capacitación | Es la posición más central de todas, pero le restaría 4 m² a la sala con más puntos de red (10) y el beneficio en distancia sería de apenas 2–3 m sobre la opción elegida. |

---

## 5. Topología física por departamento

### 5.1 Topología a nivel de edificio: árbol (estrella extendida)

El edificio completo usa una topología de **árbol**, con `SW-MDF` como raíz y los ocho
switches de departamento como nodos intermedios. Cada departamento cuelga del MDF por un
único enlace troncal.

La justifiqué así: una estrella pura desde el MDF a los 48 hosts habría exigido 48 corridas
de cable convergiendo en un solo punto —una canalización enorme y un switch de 48 puertos
saturado desde el día uno—. El árbol reduce el troncal a **8 corridas** y deja cada
departamento con su propio dominio de cableado, que se puede intervenir sin tocar a los demás.

El precio de esa decisión es que el switch de departamento es un **punto único de falla** para
su área, y el MDF lo es para todo el edificio. Lo asumo porque el enunciado pide un único
switch principal, y porque una malla o un anillo redundante multiplicaría el costo de
cableado sin que el perfil de una empresa de desarrollo de software de este tamaño lo
justifique.

### 5.2 Topología dentro de cada departamento: estrella

| Departamento | Hosts | Topología | Justificación |
|---|---:|---|---|
| Recepción | 4 | Estrella | Pocos hosts, pero uno es un servidor de recepción. La estrella aísla fallas: un cable dañado en una PC no afecta al servidor. |
| Legal | 4 | Estrella | Segmento pequeño y de baja criticidad. La estrella es lo más barato que ofrece aislamiento por puerto. |
| Recursos Humanos | 8 | Estrella | Maneja datos personales; el aislamiento por puerto facilita aplicar control de acceso más adelante sin recablear. |
| Sala de Capacitación | 10 | Estrella | Es el segmento con más hosts y el de mayor rotación de equipo. La estrella permite agregar o quitar puestos sin interrumpir a los demás. |
| Dirección General | 4 | Estrella | Segmento pequeño pero sensible. Aislamiento por puerto y capacidad sobrada para crecer. |
| Diseño e Innovación | 8 | Estrella | Transferencias de archivos grandes entre estaciones. La estrella da ancho de banda dedicado por puerto, cosa que un bus o un anillo no ofrecen. |
| Backend | 7 | Estrella | Incluye un servidor de aplicaciones. Aislamiento por puerto y ancho de banda dedicado. |
| Data Center | 3 | Estrella | Es el segmento más crítico. Aunque son solo 3 servidores, cada uno debe tener su camino independiente al switch. |

**Por qué estrella en todos los casos y no otra cosa.** Evalué tres alternativas por segmento:

- **Bus.** Descartado sin excepción. Un solo medio compartido significa un dominio de colisión
  común, un punto de falla que tumba todo el segmento y dificultad para localizar averías.
  Está además fuera de uso en cableado estructurado moderno.
- **Anillo.** Aporta tolerancia a fallos, pero exige el doble de cable y hardware que lo
  soporte. Para segmentos de 3 a 10 hosts en un edificio de un nivel, el costo no se paga.
- **Malla.** Solo tendría sentido en el Data Center, donde la criticidad es alta. La descarté
  porque con 3 servidores conectados a un mismo switch, una malla entre ellos no elimina el
  punto único de falla real, que es el switch. La forma correcta de proteger ese segmento sería
  un segundo switch con enlaces redundantes, y eso choca con el requisito de un único switch
  principal. Lo dejo anotado como mejora futura en la sección 19.

La estrella gana en los ocho casos por la misma razón de fondo: **es la única topología en la
que la falla de un cable afecta a un solo host**, y es la que asumen los estándares de cableado
estructurado TIA/EIA-568, que exigen precisamente un enlace permanente independiente por cada
punto de red.

---

## 6. Puntos de red y tomas

Definí **48 puntos de red**, uno por dispositivo. Los agrupé en **23 tomas** siguiendo el
criterio de **dos puertos por placa** para puestos de trabajo contiguos, con dos excepciones
de tres puertos donde los equipos están físicamente agrupados.

**Por qué tomas dobles y no unitarias.** Una placa doble cuesta prácticamente lo mismo que
una unitaria, y la mano de obra de instalar una caja se paga una sola vez. Además deja un
puerto disponible cuando un puesto crece a dos dispositivos (por ejemplo, una PC y un teléfono
IP). Las tomas unitarias solo tendrían sentido en puestos aislados, y en este plano no hay
ninguno.

**Por qué dos tomas triples.** En el Data Center los tres servidores están montados juntos y
una sola placa de tres puertos evita tres cajas separadas en el mismo tramo de muro. En Backend,
tres equipos (`PC-BACK-05`, `LAP-BACK-01` y `SRV-BACK-01`) comparten la fila inferior del
departamento, y la placa triple sobre el muro sur les queda a menos de 7 m.

| Toma | Departamento | Posición en planta (x, y m) | Tipo | Puertos | Puntos de red que aloja |
|---|---|---|---|---|---|
| `T-REC-01` | Recepción | (4.00, 6.00) | Doble | 2 | `Recepcion-PR01`, `Recepcion-PR02` |
| `T-REC-02` | Recepción | (11.50, 6.00) | Doble | 2 | `Recepcion-PR03`, `Recepcion-PR04` |
| `T-LEG-01` | Legal | (15.50, 6.00) | Doble | 2 | `Legal-PR01`, `Legal-PR02` |
| `T-LEG-02` | Legal | (21.00, 6.00) | Doble | 2 | `Legal-PR03`, `Legal-PR04` |
| `T-RRH-03` | Recursos Humanos | (3.50, 14.00) | Doble | 2 | `RRHH-PR01`, `RRHH-PR02` |
| `T-RRH-01` | Recursos Humanos | (8.00, 9.50) | Doble | 2 | `RRHH-PR03`, `RRHH-PR04` |
| `T-RRH-04` | Recursos Humanos | (6.00, 14.00) | Doble | 2 | `RRHH-PR05`, `RRHH-PR06` |
| `T-RRH-02` | Recursos Humanos | (8.00, 12.50) | Doble | 2 | `RRHH-PR07`, `RRHH-PR08` |
| `T-CAP-02` | Sala de Capacitación | (8.00, 10.50) | Doble | 2 | `Capacitacion-PR01`, `Capacitacion-PR06` |
| `T-CAP-01` | Sala de Capacitación | (12.00, 8.00) | Doble | 2 | `Capacitacion-PR02`, `Capacitacion-PR03` |
| `T-CAP-03` | Sala de Capacitación | (15.00, 9.50) | Doble | 2 | `Capacitacion-PR04`, `Capacitacion-PR05` |
| `T-CAP-05` | Sala de Capacitación | (13.00, 14.00) | Doble | 2 | `Capacitacion-PR07`, `Capacitacion-PR08` |
| `T-CAP-04` | Sala de Capacitación | (15.00, 11.50) | Doble | 2 | `Capacitacion-PR09`, `Capacitacion-PR10` |
| `T-DIR-01` | Dirección General | (18.00, 8.00) | Doble | 2 | `Direccion-PR01`, `Direccion-PR02` |
| `T-DIR-02` | Dirección General | (22.80, 10.00) | Doble | 2 | `Direccion-PR03`, `Direccion-PR04` |
| `T-DIS-01` | Diseño e Innovación | (8.00, 17.80) | Doble | 2 | `Diseno-PR01`, `Diseno-PR02` |
| `T-DIS-02` | Diseño e Innovación | (8.00, 19.60) | Doble | 2 | `Diseno-PR03`, `Diseno-PR04` |
| `T-DIS-03` | Diseño e Innovación | (3.50, 22.80) | Doble | 2 | `Diseno-PR05`, `Diseno-PR06` |
| `T-DIS-04` | Diseño e Innovación | (6.00, 22.80) | Doble | 2 | `Diseno-PR07`, `Diseno-PR08` |
| `T-BAC-01` | Backend | (19.00, 17.80) | Doble | 2 | `Backend-PR01`, `Backend-PR02` |
| `T-BAC-02` | Backend | (19.00, 19.60) | Doble | 2 | `Backend-PR03`, `Backend-PR04` |
| `T-BAC-03` | Backend | (16.50, 22.80) | Triple | 3 | `Backend-PR05`, `Backend-PR06`, `Backend-PR07` |
| `T-DAT-01` | Data Center | (22.80, 17.50) | Triple | 3 | `DataCenter-PR01`, `DataCenter-PR02`, `DataCenter-PR03` |

**Total: 23 tomas · 48 puertos.**

---

## 7. Medio de transmisión por segmento

| Segmento | Medio | Categoría | Velocidad soportada | Distancia máxima del diseño | Justificación |
|---|---|---|---|---|---|
| Horizontal: toma ↔ switch de departamento | UTP 4 pares, CM | **Cat6** | 1 Gbps a 100 m; 10 Gbps hasta 55 m | 15.0 m | Sobra para gigabit al escritorio. A 15 m incluso soportaría 10 Gbps si algún puesto lo necesitara. |
| Troncal: `SW-MDF` ↔ switch de departamento | UTP 4 pares, CM | **Cat6a** | 10 Gbps a 100 m | 22.7 m | Es el cuello de botella del edificio: por él pasa todo el tráfico de un departamento entero. |
| Área de trabajo: host ↔ toma | UTP flexible | Cat6 | 1 Gbps | 3 m | Patch cord de fábrica, con alivio de tensión. |

### 7.1 Por qué Cat6 en el horizontal

La distancia no es el factor decisivo: con enlaces de 15 m máximo, hasta Cat5e cumpliría
holgadamente para gigabit. Elegí Cat6 por **vida útil**, no por alcance. El cableado
horizontal es la parte de la instalación que más cuesta reemplazar, porque va dentro de muros
y canalizaciones; el equipo activo se cambia cada 5–7 años, pero el cable se espera que dure
15 o 20. Cat6 tiene mejor margen de ACR-F y menor diafonía que Cat5e, lo que le da recorrido
para 2.5G y 5GBASE-T sin recablear.

Descarté **Cat6a** para el horizontal por una razón concreta: su diámetro y su radio de curvatura
son mayores, encarece la instalación y ocupa más sección de canalización; con 48 corridas, ese
sobrecosto no compra ninguna capacidad que estos puestos vayan a usar.

Descarté **fibra óptica** al escritorio porque exigiría conversores en cada puesto y ninguna
estación de trabajo del diseño necesita más de 1 Gbps.

### 7.2 Por qué Cat6a en el troncal

Cada enlace troncal agrega el tráfico de un departamento completo. El caso más exigente es la
Sala de Capacitación: 10 hosts que pueden estar descargando material simultáneamente. Un uplink
de 1 Gbps para 10 puestos da una relación de sobresuscripción de 10:1, aceptable hoy pero
justo. Cat6a permite llevar ese uplink a 10 Gbps **sin tocar el cableado**, solo cambiando los
transceptores o el switch.

Con 147.2 m totales de troncal, el sobrecosto de subir de Cat6 a Cat6a es de aproximadamente
Q1,150 sobre el proyecto completo — menos del 2 % del presupuesto. Es la mejora de mayor
retorno del diseño.

### 7.3 Por qué no fibra en el troncal

Evalué fibra multimodo OM3 para los ocho enlaces troncales y la descarté por distancia:
**el troncal más largo mide 22.7 m**. La fibra empieza a justificarse cuando se superan los
100 m del cobre, o cuando hay que atravesar zonas con interferencia electromagnética severa o
diferencias de potencial de tierra entre edificios. Ninguna de esas condiciones se da aquí:
es un solo nivel, una sola acometida eléctrica y distancias cortas.

Además, la fibra exigiría un ODF en el MDF, transceptores SFP en los nueve switches y personal
con equipo de fusión. Ese costo no compra ninguna capacidad adicional a 22 m, porque Cat6a ya
entrega 10 Gbps a esa distancia.

---

## 8. Distancias de cableado y cálculo de bobinas

### 8.1 Método de cálculo

No medí a ojo sobre la imagen. Cada longitud sale de las coordenadas reales del plano trazado
a escala, con esta fórmula:

```
Longitud a cortar = (Recorrido ortogonal en planta + Subidas y bajadas) × 1.10 + Reserva
```

| Componente | Horizontal | Troncal | Razón |
|---|---|---|---|
| Recorrido en planta | Distancia Manhattan (‖Δx‖ + ‖Δy‖) | Ídem, siguiendo la escalerilla | El cable corre en ángulo recto por canalización, nunca en diagonal |
| Subida en el extremo de la toma | 2.6 m | 1.2 m | De la toma (0.40 m) al cielo falso (3.00 m) |
| Bajada en el extremo del equipo | 1.2 m | 1.2 m | Del cielo falso al equipo (1.80 m) |
| Holgura | 10 % | 10 % | Curvas, desvíos y tolerancia de instalación |
| Reserva de servicio | 2.0 m | 2.0 m | 1 m en cada extremo, para reterminar sin recablear |

Uso distancia **Manhattan** y no línea recta justamente porque el cable no viaja en diagonal:
sube al cielo falso, recorre la escalerilla en ángulo recto y baja. Calcular con línea recta
habría subestimado el material.

### 8.2 Cableado horizontal por departamento

#### Recepción — `SW-RECEPCION`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `Recepcion-PR01` | PC-REC-01 | `T-REC-01` | `FastEthernet0/1` | 6.5 | 13.3 |
| `Recepcion-PR02` | PC-REC-02 | `T-REC-01` | `FastEthernet0/2` | 6.5 | 13.3 |
| `Recepcion-PR03` | LAP-REC-01 | `T-REC-02` | `FastEthernet0/3` | 8.0 | 15.0 |
| `Recepcion-PR04` | SRV-REC-01 | `T-REC-02` | `FastEthernet0/4` | 8.0 | 15.0 |
| | | | **Subtotal Recepción** | | **56.6** |

#### Legal — `SW-LEGAL`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `Legal-PR01` | PC-LEG-01 | `T-LEG-01` | `FastEthernet0/1` | 6.0 | 12.8 |
| `Legal-PR02` | PC-LEG-02 | `T-LEG-01` | `FastEthernet0/2` | 6.0 | 12.8 |
| `Legal-PR03` | PC-LEG-03 | `T-LEG-02` | `FastEthernet0/3` | 6.5 | 13.3 |
| `Legal-PR04` | LAP-LEG-01 | `T-LEG-02` | `FastEthernet0/4` | 6.5 | 13.3 |
| | | | **Subtotal Legal** | | **52.2** |

#### Recursos Humanos — `SW-RRHH`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `RRHH-PR01` | PC-RRHH-01 | `T-RRH-03` | `FastEthernet0/1` | 5.9 | 12.7 |
| `RRHH-PR02` | PC-RRHH-02 | `T-RRH-03` | `FastEthernet0/2` | 5.9 | 12.7 |
| `RRHH-PR03` | PC-RRHH-03 | `T-RRH-01` | `FastEthernet0/3` | 4.5 | 11.1 |
| `RRHH-PR04` | PC-RRHH-04 | `T-RRH-01` | `FastEthernet0/4` | 4.5 | 11.1 |
| `RRHH-PR05` | PC-RRHH-05 | `T-RRH-04` | `FastEthernet0/5` | 7.0 | 13.9 |
| `RRHH-PR06` | PC-RRHH-06 | `T-RRH-04` | `FastEthernet0/6` | 7.0 | 13.9 |
| `RRHH-PR07` | LAP-RRHH-01 | `T-RRH-02` | `FastEthernet0/7` | 7.5 | 14.4 |
| `RRHH-PR08` | LAP-RRHH-02 | `T-RRH-02` | `FastEthernet0/8` | 7.5 | 14.4 |
| | | | **Subtotal Recursos Humanos** | | **104.2** |

#### Sala de Capacitación — `SW-CAPACITACION`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `Capacitacion-PR01` | PC-CAP-01 | `T-CAP-02` | `FastEthernet0/1` | 5.2 | 11.9 |
| `Capacitacion-PR02` | PC-CAP-02 | `T-CAP-01` | `FastEthernet0/2` | 1.3 | 7.6 |
| `Capacitacion-PR03` | PC-CAP-03 | `T-CAP-01` | `FastEthernet0/3` | 1.3 | 7.6 |
| `Capacitacion-PR04` | PC-CAP-04 | `T-CAP-03` | `FastEthernet0/4` | 4.2 | 10.8 |
| `Capacitacion-PR05` | PC-CAP-05 | `T-CAP-03` | `FastEthernet0/5` | 4.2 | 10.8 |
| `Capacitacion-PR06` | PC-CAP-06 | `T-CAP-02` | `FastEthernet0/6` | 5.2 | 11.9 |
| `Capacitacion-PR07` | PC-CAP-07 | `T-CAP-05` | `FastEthernet0/7` | 6.7 | 13.6 |
| `Capacitacion-PR08` | LAP-CAP-01 | `T-CAP-05` | `FastEthernet0/8` | 6.7 | 13.6 |
| `Capacitacion-PR09` | LAP-CAP-02 | `T-CAP-04` | `FastEthernet0/9` | 6.2 | 13.0 |
| `Capacitacion-PR10` | LAP-CAP-03 | `T-CAP-04` | `FastEthernet0/10` | 6.2 | 13.0 |
| | | | **Subtotal Sala de Capacitación** | | **113.8** |

#### Dirección General — `SW-DIRECCION`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `Direccion-PR01` | PC-DIR-01 | `T-DIR-01` | `FastEthernet0/1` | 1.8 | 8.2 |
| `Direccion-PR02` | PC-DIR-02 | `T-DIR-01` | `FastEthernet0/2` | 1.8 | 8.2 |
| `Direccion-PR03` | PC-DIR-03 | `T-DIR-02` | `FastEthernet0/3` | 5.0 | 11.7 |
| `Direccion-PR04` | LAP-DIR-01 | `T-DIR-02` | `FastEthernet0/4` | 5.0 | 11.7 |
| | | | **Subtotal Dirección General** | | **39.8** |

#### Diseño e Innovación — `SW-DISENO`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `Diseno-PR01` | PC-DIS-01 | `T-DIS-01` | `FastEthernet0/1` | 4.7 | 11.4 |
| `Diseno-PR02` | PC-DIS-02 | `T-DIS-01` | `FastEthernet0/2` | 4.7 | 11.4 |
| `Diseno-PR03` | PC-DIS-03 | `T-DIS-02` | `FastEthernet0/3` | 6.5 | 13.3 |
| `Diseno-PR04` | PC-DIS-04 | `T-DIS-02` | `FastEthernet0/4` | 6.5 | 13.3 |
| `Diseno-PR05` | LAP-DIS-01 | `T-DIS-03` | `FastEthernet0/5` | 6.6 | 13.4 |
| `Diseno-PR06` | LAP-DIS-02 | `T-DIS-03` | `FastEthernet0/6` | 6.6 | 13.4 |
| `Diseno-PR07` | LAP-DIS-03 | `T-DIS-04` | `FastEthernet0/7` | 7.7 | 14.7 |
| `Diseno-PR08` | SRV-DIS-01 | `T-DIS-04` | `FastEthernet0/8` | 7.7 | 14.7 |
| | | | **Subtotal Diseño e Innovación** | | **105.6** |

#### Backend — `SW-BACKEND`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `Backend-PR01` | PC-BACK-01 | `T-BAC-01` | `FastEthernet0/1` | 4.4 | 11.0 |
| `Backend-PR02` | PC-BACK-02 | `T-BAC-01` | `FastEthernet0/2` | 4.4 | 11.0 |
| `Backend-PR03` | PC-BACK-03 | `T-BAC-02` | `FastEthernet0/3` | 6.2 | 13.0 |
| `Backend-PR04` | PC-BACK-04 | `T-BAC-02` | `FastEthernet0/4` | 6.2 | 13.0 |
| `Backend-PR05` | PC-BACK-05 | `T-BAC-03` | `FastEthernet0/5` | 6.9 | 13.8 |
| `Backend-PR06` | LAP-BACK-01 | `T-BAC-03` | `FastEthernet0/6` | 6.9 | 13.8 |
| `Backend-PR07` | SRV-BACK-01 | `T-BAC-03` | `FastEthernet0/7` | 6.9 | 13.8 |
| | | | **Subtotal Backend** | | **89.4** |

#### Data Center — `SW-DATACENTER`

| Etiqueta del cable | Equipo servido | Toma | Puerto del switch | Recorrido en planta (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `DataCenter-PR01` | SRV-DC-01 | `T-DAT-01` | `FastEthernet0/1` | 2.2 | 8.6 |
| `DataCenter-PR02` | SRV-DC-02 | `T-DAT-01` | `FastEthernet0/2` | 2.2 | 8.6 |
| `DataCenter-PR03` | SRV-DC-03 | `T-DAT-01` | `FastEthernet0/3` | 2.2 | 8.6 |
| | | | **Subtotal Data Center** | | **25.8** |

### 8.3 Cableado troncal

Los ocho troncales siguen la escalerilla, no la línea recta. Recepción y Legal están al norte
del Hall Central, así que su recorrido baja al ramal EM-01, cruza por el riser EM-03 y llega al
MDF por EM-02.

| Etiqueta | Enlace | Puerto en el switch de depto. | Puerto en `SW-MDF` | Recorrido por escalerilla (m) | Longitud a cortar (m) |
|---|---|---|---|---|---|
| `MDF-Recepcion` | `SW-MDF` ↔ `SW-RECEPCION` | `GigabitEthernet0/1` | `FastEthernet0/1` | 22.7 | 29.6 |
| `MDF-Legal` | `SW-MDF` ↔ `SW-LEGAL` | `GigabitEthernet0/1` | `FastEthernet0/2` | 17.3 | 23.7 |
| `MDF-RRHH` | `SW-MDF` ↔ `SW-RRHH` | `GigabitEthernet0/1` | `FastEthernet0/3` | 15.8 | 22.0 |
| `MDF-Capacitacion` | `SW-MDF` ↔ `SW-CAPACITACION` | `GigabitEthernet0/1` | `FastEthernet0/4` | 8.5 | 14.0 |
| `MDF-Direccion` | `SW-MDF` ↔ `SW-DIRECCION` | `GigabitEthernet0/1` | `FastEthernet0/5` | 12.0 | 17.8 |
| `MDF-Diseno` | `SW-MDF` ↔ `SW-DISENO` | `GigabitEthernet0/1` | `FastEthernet0/6` | 10.9 | 16.6 |
| `MDF-Backend` | `SW-MDF` ↔ `SW-BACKEND` | `GigabitEthernet0/1` | `FastEthernet0/7` | 3.6 | 8.6 |
| `MDF-DataCenter` | `SW-MDF` ↔ `SW-DATACENTER` | `GigabitEthernet0/1` | `FastEthernet0/8` | 9.3 | 14.9 |
| | | | **Subtotal troncal** | | **147.2** |

### 8.4 Verificación contra el límite de 90 m

TIA/EIA-568 limita el **enlace permanente** horizontal a 90 m, más 10 m de patch cords en los
extremos (100 m de canal total).

| Métrica | Valor | Margen sobre el límite |
|---|---|---|
| Enlace horizontal más largo | **15.0 m** (`Recepcion-PR03` y `Recepcion-PR04`) | 83 % de margen |
| Enlace horizontal más corto | 7.6 m | — |
| Promedio de los 48 horizontales | 12.2 m | — |
| Troncal más largo | 29.6 m (`MDF-Recepcion`) | 67 % de margen |

Ningún enlace se acerca al límite. Esto valida la ubicación del MDF: no hay un solo punto de
red en riesgo de atenuación por distancia.

### 8.5 Cálculo de bobinas

| Tipo de cable | Longitud requerida | Bobinas de 305 m | Longitud adquirida | Sobrante |
|---|---:|---:|---:|---:|
| UTP Cat6 (horizontal) | 587.0 m | **2** | 610 m | 23.0 m |
| UTP Cat6a (troncal) | 147.2 m | **1** | 305 m | 157.8 m |
| **Total** | **734.2 m** | **3** | **915 m** | **180.8 m** |

**Total a adquirir: 3 bobinas de 305 m** — dos de Cat6 y una de Cat6a.

Dos observaciones honestas sobre estas cifras:

- El sobrante de Cat6 es de apenas 23 m sobre 610 m, es decir un **3.8 % de holgura**. Es un
  margen ajustado: un solo error de corte en dos o tres corridas obligaría a comprar una tercera
  bobina. Recomiendo adquirir **3 bobinas de Cat6** en la compra real y dejar la tercera como
  reserva de obra e inventario de mantenimiento. En el presupuesto de la sección 18 mantengo
  las 2 bobinas que arroja el cálculo, y señalo la tercera como partida opcional.
- El sobrante de Cat6a es grande (157.8 m) porque 147.2 m no llega ni a media bobina, y las
  bobinas no se venden fraccionadas. Ese remanente no se pierde: cubre ampliaciones futuras
  de troncal, que es exactamente el tipo de cable que uno no quiere tener que comprar de
  urgencia.

### 8.6 Compra directa de materiales frente a proveedor integral

Con 734 m de cable, 23 tomas y 56 enlaces por certificar, este proyecto está en la frontera
entre las dos modalidades. Mi recomendación es **un esquema mixto**:

| Partida | Modalidad | Razón |
|---|---|---|
| Cable, jacks, faceplates, canaleta | Compra directa | Son insumos de catálogo, con precio de lista y sin valor agregado del integrador. |
| Escalerilla y su instalación | Proveedor | Requiere anclaje a losa, herramienta y cumplimiento de normas de sujeción. |
| Ponchado, certificación y etiquetado | Proveedor | La certificación Cat6/Cat6a exige un equipo de campo que cuesta más que todo el proyecto. Sin ella no hay garantía extendida del fabricante. |
| Equipo activo | Compra directa a distribuidor autorizado | Para conservar garantía y soporte de fábrica. |

Contratar todo con un integrador único subiría el costo entre un 15 % y un 25 %, pero
concentraría la responsabilidad en un solo contrato. Para un edificio de un nivel y 48 puntos,
el esquema mixto da mejor relación costo-riesgo.

---

## 9. Canalización

**Solución elegida: escalerilla metálica abierta de 200 × 60 mm**, montada sobre el cielo falso
a 3.00 m, en tres ramales:

| Ramal | Recorrido | Longitud | Sirve a |
|---|---|---:|---|
| **EM-01** | Hall Central, de este a oeste | 20.5 m | Recepción y Legal |
| **EM-02** | Pasillo Secundario, de este a oeste | 20.5 m | RRHH, Capacitación, Dirección, Diseño, Backend y Data Center; aloja el MDF |
| **EM-03** | Riser vertical sobre el tabique Capacitación / Dirección | 9.0 m | Une EM-01 con EM-02 |
| | **Total** | **50.0 m** | |

Las bajadas de la escalerilla a cada toma van en **canaleta plástica de 40 × 25 mm** adosada
al muro, unos 70 m en total.

### 9.1 Por qué escalerilla abierta y no cerrada ni tubería

| Opción | Por qué la elegí o descarté |
|---|---|
| **Escalerilla abierta** ✅ | Es la que elegí. Permite agregar o retirar corridas sin desmontar tapas, disipa calor, y su costo por metro es el más bajo de las tres. Al ir sobre cielo falso, queda oculta a la vista y protegida del contacto accidental. |
| Escalerilla cerrada (con tapa) ❌ | Protege más del polvo, pero cuesta entre 30 % y 40 % más y obliga a desmontar tramos para cualquier cambio. En un edificio de oficinas con cielo falso no hay agresión ambiental que la justifique. |
| Tubería EMT ❌ | Ofrece la mejor protección mecánica, pero llenarla al 40 % —el máximo que permite la norma— con 48 corridas exigiría varias líneas paralelas de diámetro grande, y cada cambio futuro implicaría reponer todo el tramo. Es la peor opción para escalabilidad. |
| Bandeja tipo malla ⚠️ | Habría sido una alternativa válida y de costo similar. La descarté porque la escalerilla tiene mejor capacidad de carga en tramos largos sin soporte intermedio, y EM-01 y EM-02 miden 20.5 m cada uno. |

### 9.2 Ocupación de la canalización

El tramo más cargado es EM-02 entre el MDF y el extremo oeste, donde conviven los troncales de
seis departamentos. Un cable Cat6 tiene unos 6 mm de diámetro y uno Cat6a unos 8 mm. En el
peor punto coinciden 6 troncales Cat6a, lo que ocupa aproximadamente 300 mm² sobre una sección
útil de 12,000 mm² de la escalerilla: **menos del 3 % de ocupación**. Hay espacio de sobra para
crecer, que es justamente el argumento de escalabilidad de esta solución.

---

## 10. Equipo activo y dimensionamiento

### 10.1 Justificación de cada elemento

| Elemento | Función en el flujo de conexión | Por qué es necesario |
|---|---|---|
| **Switch de departamento** | Concentra los puntos de red de su área y entrega un puerto dedicado a cada host | Sin él, cada host necesitaría una corrida hasta el MDF: 48 cables convergiendo en un punto en vez de 8 |
| **Switch principal (`SW-MDF`)** | Interconecta los ocho departamentos | Es la raíz del árbol; sin él, los departamentos quedan aislados entre sí |
| **Patch panel de departamento** | Termina el cableado horizontal permanente | Permite que el cable rígido nunca se manipule: los cambios se hacen con patch cords. Sin él, cada movimiento desgasta el conector del cable de obra |
| **Patch panel del MDF** | Termina los troncales y administra el campo central | Da un punto único de administración y prueba para todo el edificio |
| **Organizador de cables** | Guía los patch cords dentro del gabinete | Evita que el peso de los cables fuerce los conectores y mantiene el radio de curvatura |
| **UPS** | Respalda el equipo activo ante corte eléctrico | Sin él, un corte deja sin red al edificio entero e implica reinicios abruptos que dañan la memoria flash de los switches |

### 10.2 Dimensionamiento del patch panel y del switch del edificio

La regla del enunciado es: el patch panel se dimensiona según la cantidad total de puntos de
red, y el switch debe tener una cantidad de puertos igual o mayor.

| Concepto | Cálculo | Resultado |
|---|---|---|
| Puntos de red del edificio | 30 PC + 12 laptops + 6 servidores | **48** |
| Patch panel del edificio | 48 puntos → tamaño comercial inmediato | **48 puertos** |
| Switch del edificio | Debe ser ≥ patch panel | **48 puertos** |

Los patch panels de departamento se dimensionan igual, sobre sus propios puntos:

| Departamento | Puntos | Patch panel | Puertos libres |
|---|---:|---:|---:|
| Recepción | 4 | 12 p | 8 |
| Legal | 4 | 12 p | 8 |
| Recursos Humanos | 8 | 12 p | 4 |
| Sala de Capacitación | 10 | 12 p | 2 |
| Dirección General | 4 | 12 p | 8 |
| Diseño e Innovación | 8 | 12 p | 4 |
| Backend | 7 | 12 p | 5 |
| Data Center | 3 | 12 p | 9 |
| **Total** | **48** | **96 p** | **48** |

Usé 12 puertos en todos los departamentos, incluso donde sobran ocho, porque es el tamaño
comercial más pequeño que permite montaje en rack y la diferencia de precio contra uno de
menor capacidad es marginal. El resultado es un **100 % de capacidad de crecimiento** en
terminación sin comprar nada nuevo.

Los switches de departamento son de 24 puertos: el más cargado es el de Capacitación, con 10
puntos ocupados y 14 libres.

### 10.3 Nota sobre el archivo de Packet Tracer

En el archivo `.pkt` los nueve switches son **Cisco 2960-24TT**, de 24 puertos FastEthernet más
2 Gigabit. Esto merece una aclaración, porque no coincide con el dimensionamiento que acabo de
justificar:

- El catálogo de Packet Tracer **no incluye ningún switch de 48 puertos**. Los modelos de
  switch disponibles (2950-24, 2950T-24, 2960-24TT, 3560-24PS, 3650-24PS, IE-2000) llegan como
  máximo a 24 puertos de acceso más 2 uplinks.
- Por eso `SW-MDF` aparece en la simulación como un 2960-24TT usando 8 puertos FastEthernet
  para los troncales. **En la implementación real corresponde un switch de 48 puertos con
  uplinks Gigabit o SFP+**, tal como lo especifiqué en el inventario y el presupuesto.
- Consecuencia técnica de la limitación: en la simulación los troncales terminan en puertos
  FastEthernet de 100 Mbps, lo que contradice el uso de Cat6a. El diagrama documenta la
  intención de diseño; el equipo real debe tener esos ocho puertos a 1 Gbps como mínimo.

Lo dejo explícito para que quede claro que es una restricción de la herramienta de
diagramación y no un error de dimensionamiento.

---

## 11. Gabinete del MDF

**Propuesta: gabinete de pared de 12U**, con puerta de vidrio templado y cerradura, ventilación
pasiva superior y profundidad de 450 mm.

### 11.1 Cálculo de la altura necesaria

| Elemento | Altura |
|---|---:|
| Switch principal de 48 puertos | 1U |
| Patch panel de 48 puertos | 1U |
| Organizadores horizontales | 2U |
| UPS de montaje en rack | 2U |
| Bandeja de servicio / reserva | 1U |
| **Ocupación total** | **7U** |
| **Capacidad del gabinete** | **12U** |
| **Espacio libre** | **5U (42 %)** |

### 11.2 Gabinete de pared frente a rack de piso

| Criterio | Gabinete de pared 12U ✅ | Rack de piso 42U ❌ |
|---|---|---|
| Equipo a alojar | 7U — sobra espacio | 7U sobre 42U: 83 % desaprovechado |
| Espacio de piso | Cero: va sobre el muro del corredor | Necesita ~1 m² más área de maniobra, y el plano no tiene espacio ocioso |
| Seguridad | Puerta con cerradura, montado a 1.80 m, fuera de alcance casual | Igual seguridad, pero accesible desde el piso |
| Costo | Aproximadamente un tercio | Sobredimensionado para este proyecto |
| Escalabilidad | 5U libres cubren duplicar el equipo activo | Capacidad que no se va a usar |

Un rack de piso solo se justificaría si el edificio creciera a varios niveles y el MDF pasara a
concentrar equipo de distribución, servidores o UPS de mayor porte. Con 48 puntos en una sola
planta, sería comprar capacidad ociosa.

---

## 12. Respaldo de energía (UPS)

### 12.1 Estimación del consumo

| Equipo | Cantidad | Consumo unitario | Subtotal |
|---|---:|---:|---:|
| Switch de acceso 24 p (sin PoE) | 8 | ~30 W | 240 W |
| Switch principal 48 p | 1 | ~30 W | 30 W |
| **Carga total del equipo activo** | | | **270 W** |

> **Supuesto declarado:** tomé 30 W como consumo por switch, que es el orden de magnitud
> habitual de un switch de acceso de 24 puertos sin alimentación PoE. Este valor **debe
> validarse contra la hoja de datos del modelo que finalmente se adquiera**, porque un modelo
> con PoE puede multiplicar esa cifra varias veces.

### 12.2 Dimensionamiento

| Paso | Cálculo | Resultado |
|---|---|---|
| Carga real | 270 W | 270 W |
| Conversión a VA (factor de potencia 0.9) | 270 / 0.9 | 300 VA |
| Margen de crecimiento del 40 % | 300 × 1.4 | 420 VA |
| Capacidad comercial inmediata superior | — | **1500 VA / 900 W** |

**Propuesta: UPS línea interactiva de 1500 VA / 900 W**, con montaje en gabinete, regulación
automática de voltaje (AVR) y salidas con protección contra sobretensión.

Elegí 1500 VA y no 1000 VA porque la carga quedaría en el 30 % de la capacidad, y es
precisamente en ese rango donde una UPS entrega su mejor autonomía. Con 270 W sobre 900 W
disponibles, la autonomía estimada es de **25 a 30 minutos**, suficiente para que un corte
breve pase sin incidente y para apagar ordenadamente si se prolonga.

> **Supuesto declarado:** la autonomía depende de la curva de descarga del fabricante. La cifra
> anterior es una estimación típica del segmento y debe confirmarse con la hoja de datos.

Elegí **línea interactiva** y no doble conversión porque la carga es de equipo de red con
fuentes conmutadas, tolerante a la transferencia de unos milisegundos que introduce esta
topología. Una UPS de doble conversión costaría cerca del doble y su ventaja —salida
permanentemente regenerada— solo se paga con equipo sensible a microcortes.

---

## 13. Aplicación de los estándares T568A / T568B

### 13.1 Criterio aplicado

| Tipo de enlace | Dispositivos en cada extremo | Cable | Cantidad |
|---|---|---|---:|
| Horizontal | Switch (MDI-X) ↔ Host (MDI) | **Straight-through** | 48 |
| Troncal | Switch (MDI-X) ↔ Switch (MDI-X) | **Crossover** | 8 |

**La regla técnica.** Un puerto de host (`MDI`) transmite en los pines 1-2 y recibe en los 3-6.
Un puerto de switch (`MDI-X`) hace exactamente lo contrario: transmite en 3-6 y recibe en 1-2.
Es decir, **el switch ya lleva el cruce interno**. Por eso al unir un host con un switch el
cable debe ser recto: cruzarlo dos veces dejaría transmisor contra transmisor.

Cuando los dos extremos son del mismo tipo —dos switches, o dos hosts— ninguno aporta el
cruce, y hay que hacerlo en el cable: ese es el crossover.

**Aclaración sobre Auto-MDI-X.** Los switches modernos negocian el cruce automáticamente, así
que en la práctica un cable recto funcionaría también entre switches. Aun así documento y
especifico el crossover porque el enunciado pide aplicar el estándar, porque Auto-MDI-X puede
deshabilitarse por configuración, y porque un enlace troncal debe funcionar aunque la
negociación falle o se fije la velocidad manualmente.

### 13.2 Tabla de enlaces

| # | Enlace | Extremo A | Extremo B | Tipo de cable | Norma en cada extremo |
|---|---|---|---|---|---|
| 1 | `SW-RECEPCION` ↔ `PC-REC-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 2 | `SW-RECEPCION` ↔ `PC-REC-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 3 | `SW-RECEPCION` ↔ `LAP-REC-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 4 | `SW-RECEPCION` ↔ `SRV-REC-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 5 | `SW-LEGAL` ↔ `PC-LEG-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 6 | `SW-LEGAL` ↔ `PC-LEG-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 7 | `SW-LEGAL` ↔ `PC-LEG-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 8 | `SW-LEGAL` ↔ `LAP-LEG-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 9 | `SW-RRHH` ↔ `PC-RRHH-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 10 | `SW-RRHH` ↔ `PC-RRHH-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 11 | `SW-RRHH` ↔ `PC-RRHH-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 12 | `SW-RRHH` ↔ `PC-RRHH-04` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 13 | `SW-RRHH` ↔ `PC-RRHH-05` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 14 | `SW-RRHH` ↔ `PC-RRHH-06` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 15 | `SW-RRHH` ↔ `LAP-RRHH-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 16 | `SW-RRHH` ↔ `LAP-RRHH-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 17 | `SW-CAPACITACION` ↔ `PC-CAP-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 18 | `SW-CAPACITACION` ↔ `PC-CAP-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 19 | `SW-CAPACITACION` ↔ `PC-CAP-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 20 | `SW-CAPACITACION` ↔ `PC-CAP-04` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 21 | `SW-CAPACITACION` ↔ `PC-CAP-05` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 22 | `SW-CAPACITACION` ↔ `PC-CAP-06` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 23 | `SW-CAPACITACION` ↔ `PC-CAP-07` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 24 | `SW-CAPACITACION` ↔ `LAP-CAP-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 25 | `SW-CAPACITACION` ↔ `LAP-CAP-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 26 | `SW-CAPACITACION` ↔ `LAP-CAP-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 27 | `SW-DIRECCION` ↔ `PC-DIR-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 28 | `SW-DIRECCION` ↔ `PC-DIR-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 29 | `SW-DIRECCION` ↔ `PC-DIR-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 30 | `SW-DIRECCION` ↔ `LAP-DIR-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 31 | `SW-DISENO` ↔ `PC-DIS-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 32 | `SW-DISENO` ↔ `PC-DIS-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 33 | `SW-DISENO` ↔ `PC-DIS-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 34 | `SW-DISENO` ↔ `PC-DIS-04` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 35 | `SW-DISENO` ↔ `LAP-DIS-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 36 | `SW-DISENO` ↔ `LAP-DIS-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 37 | `SW-DISENO` ↔ `LAP-DIS-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 38 | `SW-DISENO` ↔ `SRV-DIS-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 39 | `SW-BACKEND` ↔ `PC-BACK-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 40 | `SW-BACKEND` ↔ `PC-BACK-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 41 | `SW-BACKEND` ↔ `PC-BACK-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 42 | `SW-BACKEND` ↔ `PC-BACK-04` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 43 | `SW-BACKEND` ↔ `PC-BACK-05` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 44 | `SW-BACKEND` ↔ `LAP-BACK-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 45 | `SW-BACKEND` ↔ `SRV-BACK-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 46 | `SW-DATACENTER` ↔ `SRV-DC-01` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 47 | `SW-DATACENTER` ↔ `SRV-DC-02` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 48 | `SW-DATACENTER` ↔ `SRV-DC-03` | Switch (MDI-X) | Host (MDI) | **Straight-through** | T568B — T568B |
| 49 | `SW-MDF` ↔ `SW-RECEPCION` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 50 | `SW-MDF` ↔ `SW-LEGAL` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 51 | `SW-MDF` ↔ `SW-RRHH` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 52 | `SW-MDF` ↔ `SW-CAPACITACION` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 53 | `SW-MDF` ↔ `SW-DIRECCION` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 54 | `SW-MDF` ↔ `SW-DISENO` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 55 | `SW-MDF` ↔ `SW-BACKEND` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |
| 56 | `SW-MDF` ↔ `SW-DATACENTER` | Switch (MDI-X) | Switch (MDI-X) | **Crossover** | T568A — T568B |

---

## 14. Disposición de pines

Adopté **T568B** como norma base de la instalación, porque es la de uso predominante en
instalaciones comerciales de la región y la que traen preconfigurados la mayoría de jacks y
patch panels del mercado local. Lo importante no es cuál de las dos se elija, sino
**mantenerla igual en los dos extremos de cada enlace permanente**: en el jack de la toma y en
el puerto del patch panel.

### 14.1 Cable straight-through (T568B en ambos extremos)

Usado en los 48 enlaces horizontales.

| Pin | Extremo A — color | Extremo B — color | Par | Función en 100BASE-TX |
|---:|---|---|---:|---|
| 1 | Blanco / Naranja | Blanco / Naranja | 2 | TX+ |
| 2 | Naranja | Naranja | 2 | TX− |
| 3 | Blanco / Verde | Blanco / Verde | 3 | RX+ |
| 4 | Azul | Azul | 1 | No usado (sí en 1000BASE-T) |
| 5 | Blanco / Azul | Blanco / Azul | 1 | No usado (sí en 1000BASE-T) |
| 6 | Verde | Verde | 3 | RX− |
| 7 | Blanco / Café | Blanco / Café | 4 | No usado (sí en 1000BASE-T) |
| 8 | Café | Café | 4 | No usado (sí en 1000BASE-T) |

Cada pin conserva su posición: el pin 1 de un extremo llega al pin 1 del otro. En 1000BASE-T
los cuatro pares transmiten y reciben simultáneamente, por eso el cable debe estar completo y
con los pares correctamente trenzados hasta el conector.

### 14.2 Cable crossover (T568A en un extremo, T568B en el otro)

Usado en los 8 enlaces troncales `SW-MDF` ↔ switch de departamento.

| Pin | Extremo A — **T568A** | Extremo B — **T568B** | Cruce |
|---:|---|---|---|
| 1 | Blanco / Verde | Blanco / Naranja | 1 ↔ 3 |
| 2 | Verde | Naranja | 2 ↔ 6 |
| 3 | Blanco / Naranja | Blanco / Verde | 3 ↔ 1 |
| 4 | Azul | Azul | 4 ↔ 4 |
| 5 | Blanco / Azul | Blanco / Azul | 5 ↔ 5 |
| 6 | Naranja | Verde | 6 ↔ 2 |
| 7 | Blanco / Café | Blanco / Café | 7 ↔ 7 |
| 8 | Café | Café | 8 ↔ 8 |

La diferencia entre las dos normas es únicamente el **intercambio del par 2 (naranja) con el
par 3 (verde)**. Los pares 1 (azul) y 4 (café) quedan igual en ambas. Ponchando T568A de un
lado y T568B del otro, el par de transmisión de un switch cae sobre el par de recepción del
otro, que es exactamente lo que se necesita entre dos dispositivos MDI-X.

> Para gigabit completo (1000BASE-T) un crossover estricto también cruzaría los pares 1 y 4.
> En la práctica el crossover de dos pares que aquí documento es el estándar de uso general y
> el que negocia correctamente con Auto-MDI-X.

---

## 15. Etiquetado de cables

### 15.1 Formato adoptado

| Tipo de cable | Formato | Ejemplo | Se coloca en |
|---|---|---|---|
| Horizontal | `[Departamento]-PR[nn]` | `Recepcion-PR01` | Ambos extremos: jack de la toma y puerto del patch panel |
| Troncal | `MDF-[Departamento]` | `MDF-Backend` | Ambos extremos: patch panel del MDF y uplink del switch de departamento |

La numeración `PR` (Punto de Red) es correlativa dentro de cada departamento y empieza en 01.

### 15.2 Tabla de etiquetado

| Departamento | Etiquetas del cableado horizontal | Etiqueta del troncal |
|---|---|---|
| Recepción | `Recepcion-PR01` … `Recepcion-PR04` (4 cables) | `MDF-Recepcion` |
| Legal | `Legal-PR01` … `Legal-PR04` (4 cables) | `MDF-Legal` |
| Recursos Humanos | `RRHH-PR01` … `RRHH-PR08` (8 cables) | `MDF-RRHH` |
| Sala de Capacitación | `Capacitacion-PR01` … `Capacitacion-PR10` (10 cables) | `MDF-Capacitacion` |
| Dirección General | `Direccion-PR01` … `Direccion-PR04` (4 cables) | `MDF-Direccion` |
| Diseño e Innovación | `Diseno-PR01` … `Diseno-PR08` (8 cables) | `MDF-Diseno` |
| Backend | `Backend-PR01` … `Backend-PR07` (7 cables) | `MDF-Backend` |
| Data Center | `DataCenter-PR01` … `DataCenter-PR03` (3 cables) | `MDF-DataCenter` |
| **Total** | **48 cables horizontales** | **8 troncales** |

El detalle cable por cable, con su toma, puerto de switch y longitud, está en las tablas de las
secciones 8.2 y 8.3.

### 15.3 Etiquetado de espacios y canalizaciones

Además de los cables, identifiqué los elementos de infraestructura:

| Elemento | Identificador | Ubicación |
|---|---|---|
| Cuarto de telecomunicaciones | `MDF` | Pasillo Secundario (13.50, 15.20) m |
| Ramal de escalerilla, Hall Central | `EM-01` | Hall Central |
| Ramal de escalerilla, Pasillo Secundario | `EM-02` | Pasillo Secundario |
| Riser vertical | `EM-03` | Tabique Capacitación / Dirección |
| Tomas de red | `T-[DEP]-[nn]` | Ver tabla de la sección 6 |

---

## 16. Comparación con el estándar TIA/EIA-606

El estándar **TIA/EIA-606** (*Administration Standard for Telecommunications Infrastructure*)
define cómo identificar y documentar formalmente toda la infraestructura: cables, terminaciones,
espacios, canalizaciones y sistemas de puesta a tierra. Establece cuatro clases de
administración (Clase 1 a Clase 4) según la complejidad de la instalación; un edificio de un
nivel con un solo cuarto de telecomunicaciones, como este, corresponde a **Clase 1**.

### 16.1 Diferencias concretas

| Aspecto | Esquema de esta práctica | Lo que exige TIA/EIA-606 |
|---|---|---|
| **1. Unicidad del identificador** | `Recepcion-PR01` es único solo dentro de su departamento. El número `PR01` se repite en los ocho departamentos y no codifica edificio, nivel ni cuarto de telecomunicaciones. | Cada identificador debe ser **único en toda la infraestructura**. En Clase 1 el formato es `f-an`, donde `f` identifica el espacio de telecomunicaciones y `an` la posición exacta en el campo de terminación — por ejemplo `1A-A07`. Así, dos edificios o dos pisos jamás producen la misma etiqueta. |
| **2. Registro documental** | Las etiquetas viven en el cable y en las tablas de este manual. No hay un registro formal que ligue cada cable con su puerto de patch panel, su puerto de switch y su historial de cambios. | Exige **registros** (*records*) que vinculen cada enlace con su terminación, su canalización, su espacio y las pruebas de certificación, con control de versiones. La etiqueta física es solo el índice; el valor está en el registro que la respalda. |
| **3. Código de colores** | No uso codificación por color. Todas las terminaciones son iguales. | Define colores normalizados por tipo de campo: **naranja** para el punto de demarcación, **verde** para conexiones de red, **azul** para horizontal al área de trabajo, **morado** para equipo común, **blanco/gris** para backbone de primer y segundo nivel. Permite identificar la función de un campo sin leer una sola etiqueta. |
| **4. Alcance de lo identificado** | Identifico cables, tomas, el MDF y los tres ramales de escalerilla. | Además exige identificar formalmente **espacios, canalizaciones, aterrizaje y unión** (TMGB / TGB), y firestops en cada penetración de muro o losa. |

### 16.2 Por qué en un entorno real se usaría el estándar completo

Con 48 puntos, un solo cuarto de telecomunicaciones y una persona que conoce la instalación
de memoria, un esquema simplificado funciona. El estándar completo se vuelve necesario cuando
desaparecen esas tres condiciones, y en un data center o un edificio corporativo desaparecen
las tres a la vez.

La razón de fondo es **la trazabilidad sin inspección física**. En una instalación de cientos o
miles de enlaces, localizar el otro extremo de un cable rastreándolo a mano es inviable: obliga
a levantar cielos falsos y abrir canalizaciones, con riesgo de dañar enlaces en servicio. Con
identificadores únicos y registros actualizados, un técnico que nunca vio la instalación
resuelve una falla o ejecuta un traslado consultando la base de datos.

A eso se suman dos consecuencias prácticas: la rotación de personal deja de ser un riesgo,
porque el conocimiento está en el registro y no en la cabeza de quien instaló; y **las garantías
extendidas de fabricante —típicamente de 20 a 25 años— exigen documentación de etiquetado y
resultados de certificación** como condición para ser válidas. Sin administración formal, no
hay garantía que reclamar.

---

## 17. Flujo de conexión end-to-end

Sigo el camino completo de `PC-CAP-03`, una estación de la Sala de Capacitación, hasta el
switch principal del edificio.

| # | Tramo | Elemento | Detalle |
|---:|---|---|---|
| 1 | Área de trabajo | `PC-CAP-03` | Tarjeta de red, puerto `MDI` |
| 2 | Patch cord | Cat6 de 3 m, T568B en ambos extremos | Del equipo al jack de la toma |
| 3 | Toma de red | `T-CAP-01`, placa doble sobre el muro norte de Capacitación, punto `Capacitacion-PR03` | Jack keystone Cat6 ponchado en T568B |
| 4 | Bajada | Canaleta 40 × 25 mm | De la toma (0.40 m) al cielo falso (3.00 m) |
| 5 | Canalización horizontal | Escalerilla `EM-02` | Recorrido en ángulo recto dentro del departamento |
| 6 | Enlace permanente | Cable `Capacitacion-PR03`, UTP Cat6, **7.6 m**, straight-through | Enlace horizontal, muy por debajo de los 90 m del estándar |
| 7 | Terminación | Patch panel Cat6 de 12 puertos de Capacitación, puerto 3 | Ponchado en T568B, igual que en la toma |
| 8 | Patch cord | Cat6 de 1 m | Del patch panel al switch |
| 9 | Equipo activo del departamento | `SW-CAPACITACION`, puerto `FastEthernet0/3` | Puerto `MDI-X` |
| 10 | Uplink | `SW-CAPACITACION`, puerto `GigabitEthernet0/1` | Salida del departamento |
| 11 | Enlace troncal | Cable `MDF-Capacitacion`, UTP Cat6a, **14.0 m**, crossover T568A/T568B | Switch contra switch, ambos `MDI-X` |
| 12 | Canalización troncal | Escalerilla `EM-02` | Del departamento al MDF por el Pasillo Secundario |
| 13 | Terminación central | Patch panel de 48 puertos del MDF | Campo de administración del edificio |
| 14 | Patch cord | Cat6a de 1 m | Del patch panel al switch principal |
| 15 | Equipo activo central | `SW-MDF`, puerto `FastEthernet0/4` | Raíz del árbol |

**Longitud total del canal:** 3 m + 7.6 m + 1 m = **11.6 m** en el segmento horizontal, contra
los 100 m que permite el estándar para el canal completo. Y **14.0 m + 1 m = 15.0 m** en el
segmento troncal.

Este recorrido muestra los dos cambios de norma que importan en Capa 1: el cable horizontal es
recto porque une un host (`MDI`) con un switch (`MDI-X`), y el troncal es cruzado porque une dos
switches, ambos `MDI-X`.

---

## 18. Presupuesto estimado

> **Todos los precios son estimaciones de mercado en quetzales (GTQ) para efectos de
> planificación.** No corresponden a una cotización formal y deben validarse con proveedores
> antes de cualquier compra. Los marco explícitamente como supuesto porque el enunciado pide un
> presupuesto estimado, no una cotización.

| Categoría | Descripción | Unidad | Cant. | P. unitario (Q) | Subtotal (Q) |
|---|---|---|---:|---:|---:|
| Equipo activo | Switch de acceso gestionable 24 puertos 10/100 + 2 uplinks Gigabit | unidad | 8 | 3,200.00 | 25,600.00 |
| Equipo activo | Switch principal del MDF, 48 puertos + uplinks Gigabit | unidad | 1 | 7,500.00 | 7,500.00 |
| Administración | Patch panel Cat6 de 12 puertos (uno por departamento) | unidad | 8 | 450.00 | 3,600.00 |
| Administración | Patch panel Cat6 de 48 puertos (MDF) | unidad | 1 | 1,300.00 | 1,300.00 |
| Administración | Gabinete de pared 12U con puerta de vidrio y cerradura | unidad | 1 | 2,900.00 | 2,900.00 |
| Administración | Organizador horizontal de cables 1U | unidad | 2 | 180.00 | 360.00 |
| Energía | UPS línea interactiva 1500 VA / 900 W, montaje en gabinete | unidad | 1 | 2,600.00 | 2,600.00 |
| Cable | Bobina UTP Cat6 CM, 305 m (cableado horizontal) | bobina | 2 | 1,450.00 | 2,900.00 |
| Cable | Bobina UTP Cat6a CM, 305 m (cableado troncal) | bobina | 1 | 2,600.00 | 2,600.00 |
| Terminación | Jack keystone Cat6 T568A/B | unidad | 48 | 42.00 | 2,016.00 |
| Terminación | Faceplate doble con tapa | unidad | 21 | 32.00 | 672.00 |
| Terminación | Faceplate triple con tapa | unidad | 2 | 45.00 | 90.00 |
| Terminación | Caja de montaje superficial para faceplate | unidad | 23 | 22.00 | 506.00 |
| Patch cords | Patch cord Cat6 de 3 m (área de trabajo: host a toma) | unidad | 48 | 58.00 | 2,784.00 |
| Patch cords | Patch cord Cat6 de 1 m (patch panel a switch de departamento) | unidad | 48 | 38.00 | 1,824.00 |
| Patch cords | Patch cord Cat6a de 1 m (patch panel del MDF a SW-MDF) | unidad | 8 | 75.00 | 600.00 |
| Canalización | Escalerilla metálica abierta galvanizada 200 x 60 mm | metro | 50 | 165.00 | 8,250.00 |
| Canalización | Accesorios de escalerilla: curvas, uniones, soportes y anclajes | lote | 1 | 1,800.00 | 1,800.00 |
| Canalización | Canaleta plástica 40 x 25 mm para bajadas a tomas | metro | 70 | 28.00 | 1,960.00 |
| Identificación | Material de etiquetado: etiquetas autolaminadas y marcadores | lote | 1 | 650.00 | 650.00 |
| Pruebas | Certificación de enlaces permanentes Cat6/Cat6a | enlace | 56 | 45.00 | 2,520.00 |
| | **Subtotal de materiales y equipo** | | | | **73,032.00** |
| Servicio | Mano de obra de instalación y certificación (30 % de materiales) | global | 1 | 21,910.00 | 21,910.00 |
| | **Subtotal antes de impuestos** | | | | **94,942.00** |
| Impuesto | IVA 12 % | | | | 11,393.00 |
| | **TOTAL ESTIMADO** | | | | **106,335.00** |

### 18.1 Distribución del costo

| Categoría | Monto (Q) | Porcentaje |
|---|---:|---:|
| Equipo activo | 33,100.00 | 45.3 % |
| Canalización | 12,010.00 | 16.4 % |
| Administración (gabinete y patch panels) | 8,160.00 | 11.2 % |
| Cable | 5,500.00 | 7.5 % |
| Patch cords | 5,208.00 | 7.1 % |
| Terminación | 3,284.00 | 4.5 % |
| Energía | 2,600.00 | 3.6 % |
| Pruebas | 2,520.00 | 3.5 % |
| Identificación | 650.00 | 0.9 % |
| **Subtotal de materiales** | **73,032.00** | **100 %** |

Casi la mitad del presupuesto es equipo activo, y la canalización es el segundo rubro. Esa
proporción es la esperable en un proyecto de un solo nivel con distancias cortas: el cable en
sí apenas representa un 7.5 %, porque hay poco metraje.

### 18.2 Partidas opcionales

| Partida | Monto (Q) | Cuándo conviene |
|---|---:|---|
| Tercera bobina de Cat6 como reserva de obra | 1,450.00 | Recomendada — el margen calculado es de solo 3.8 % (ver sección 8.5) |
| Segunda UPS para redundancia N+1 | 2,600.00 | Si la operación no tolera ventanas de mantenimiento de la UPS |
| Switch de repuesto en frío | 3,200.00 | Reduce el tiempo de reposición ante falla de un switch de departamento |

---

## 19. Consideraciones de escalabilidad futura

| Recurso | Capacidad instalada | En uso | Margen |
|---|---:|---:|---:|
| Puertos en switches de departamento | 192 | 48 | **75 % libre** |
| Puertos en patch panels de departamento | 96 | 48 | **50 % libre** |
| Puertos en el switch del MDF | 48 | 8 | **83 % libre** |
| Unidades de rack en el gabinete | 12U | 7U | **42 % libre** |
| Sección útil de la escalerilla EM-02 | ~12,000 mm² | ~300 mm² | **97 % libre** |
| Capacidad de la UPS | 900 W | 270 W | **70 % libre** |

El diseño admite **duplicar la cantidad de puntos de red sin cambiar la infraestructura
pasiva**: hay puertos, espacio en canalización, unidades de rack y respaldo eléctrico de sobra.
El único elemento que habría que tocar son los patch panels de Capacitación y de RRHH, que
quedan con 2 y 4 puertos libres respectivamente.

Tres mejoras que dejo identificadas para una fase posterior:

1. **Redundancia del MDF.** Hoy `SW-MDF` es punto único de falla para todo el edificio. Un
   segundo switch en el gabinete, con enlaces redundantes hacia los departamentos críticos
   (Data Center y Backend) y protocolo de árbol de expansión, eliminaría ese riesgo. Las 5U
   libres del gabinete ya lo permiten.
2. **Uplinks a 10 Gbps.** El Cat6a del troncal ya soporta 10GBASE-T. Basta cambiar el equipo
   activo cuando el tráfico lo pida; el cableado no se toca.
3. **Cobertura inalámbrica.** El plano no incluye puntos de acceso. Si se agregan, la
   escalerilla ya llega a los dos corredores, que es donde conviene montarlos, y habría que
   prever switches con PoE — lo cual cambiaría de raíz el cálculo de UPS de la sección 12.

---

## 20. Supuestos y limitaciones

Declaro explícitamente lo que asumí, para que se pueda auditar el diseño:

| # | Supuesto | Impacto si cambia |
|---:|---|---|
| 1 | Altura de cielo falso: 3.00 m; tomas a 0.40 m; equipo activo a 1.80 m | Cambian las longitudes de subida y bajada, y con ellas el metraje total |
| 2 | Consumo de 30 W por switch, sin PoE | Con switches PoE la UPS quedaría subdimensionada |
| 3 | Precios de mercado estimados en GTQ, sin cotización formal | El total del presupuesto varía |
| 4 | Holgura de instalación del 10 % y reserva de servicio de 2 m por corrida | Menos holgura reduce el metraje, pero deja sin margen para reterminar |
| 5 | Los muros interiores admiten canaleta superficial | Si el cableado debe ir embutido, cambia el costo de mano de obra |
| 6 | Distribución de los 42 equipos entre departamentos según lo permite el enunciado | Es una decisión propia, no impuesta por el plano |

**Limitación de la herramienta.** El archivo `.pkt` usa switches Cisco 2960-24TT porque el
catálogo de Packet Tracer no ofrece modelos de 48 puertos. El dimensionamiento correcto para la
implementación real es el de la sección 10.2. Lo detallo en la sección 10.3.

---

## 21. Referencias

- ANSI/TIA-568.2-D — *Balanced Twisted-Pair Telecommunications Cabling and Components Standard*
- ANSI/TIA-569 — *Telecommunications Pathways and Spaces*
- ANSI/TIA-606-C — *Administration Standard for Telecommunications Infrastructure*
- ANSI/TIA-607 — *Generic Telecommunications Bonding and Grounding*
- Odom, Wendell (2019). *CCNA 200-301 Official Cert Guide, Vol. 1*. Cisco Press.
- Catálogo de infraestructura de redes Panduit, edición 2025 (Latinoamérica)
- Catálogo Siemon 2022
