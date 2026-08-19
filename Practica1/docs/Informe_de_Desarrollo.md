# Informe de Desarrollo

**Práctica 1 · QuetzalDev S.A. — Diseño físico de red**
Redes de Computadoras 1 · Segundo Semestre 2026

| | |
|---|---|
| **Estudiante** | Alex Ricardo Castañeda Rodríguez |
| **Carnet** | 202300476 |
| **Plano asignado** | `Plano4.png` — edificio de un nivel, 22 × 22 m |

---

## 1. Cómo abordé el problema

El enunciado pedía diseñar, no configurar. Esa distinción marcó todo el trabajo: en ningún
momento asigné direcciones IP, VLANs ni protocolos, porque el ejercicio es de Capa 1. Lo que
tenía que resolver era dónde va cada cosa físicamente y por qué.

Trabajé en cuatro etapas:

1. **Leer el plano y convertirlo en números.** Antes de decidir nada, traduje el plano
   arquitectónico a coordenadas. Definí una escala de 100 unidades de lienzo por metro y ubiqué
   los once espacios con sus dimensiones exactas. Esto fue lo más importante del proceso,
   aunque no se vea en el resultado: sin coordenadas, todos los cálculos de distancia habrían
   sido estimaciones a ojo sobre una imagen.

2. **Distribuir los 48 equipos.** El enunciado fija cuántos hosts lleva cada departamento pero
   deja libre el reparto de los 30 PC, 12 laptops y 6 servidores. Asigné laptops a los espacios
   de trabajo más móviles (Capacitación, Diseño) y PC de escritorio a los puestos fijos.

3. **Decidir topología, medio y canalización.** Cada decisión la tomé comparando al menos dos
   alternativas y descartando con un motivo medible, no por costumbre.

4. **Documentar y verificar.** Recalculé todas las distancias desde las coordenadas y comprobé
   que ningún enlace se acercara al límite de 90 m del estándar.

---

## 2. Criterios de selección de topología

La pregunta de fondo era si convenía una estrella pura desde el cuarto de telecomunicaciones o
un árbol con switches por departamento.

Descarté la estrella pura por una razón concreta: implicaba **48 corridas de cable convergiendo
en un único punto**. Eso significa una canalización mucho mayor en el tramo final, un patch panel
de 48 puertos con toda la instalación dependiendo de él, y un metraje total sensiblemente mayor,
porque cada host pagaría la distancia completa hasta el MDF en lugar de la distancia a su switch
de departamento.

El árbol reduce el troncal a **8 corridas** y deja cada departamento como un dominio de cableado
independiente: se puede intervenir un área sin tocar las demás.

Dentro de cada departamento elegí estrella en los ocho casos. No fue una decisión automática:
evalué bus, anillo y malla. El bus quedó descartado de entrada porque un solo medio compartido
es un punto de falla total del segmento. El anillo aporta tolerancia a fallos pero duplica el
cable y exige hardware que lo soporte, algo que no se paga en segmentos de 3 a 10 hosts. La malla
la consideré solo para el Data Center, y al analizarla descubrí que no resolvía nada: con tres
servidores colgando del mismo switch, el punto único de falla real es el switch, no los enlaces.

El argumento decisivo a favor de la estrella es que **es la única topología en la que un cable
dañado afecta a un solo host**, y es la que asumen los estándares de cableado estructurado, que
exigen un enlace permanente independiente por punto de red.

Asumo conscientemente el costo de esta decisión: el switch de departamento es punto único de
falla para su área y el MDF lo es para el edificio. Lo dejé identificado como mejora futura en
el manual, con una propuesta concreta de redundancia.

---

## 3. Criterios de selección del medio de transmisión

### Cableado horizontal

Elegí **UTP Cat6**. Lo interesante de este caso es que la distancia **no** fue el factor
decisivo: con el enlace más largo en 15 m, hasta Cat5e habría cumplido de sobra para gigabit.

El criterio real fue la **vida útil**. El cableado horizontal es la parte más cara de reemplazar
porque va dentro de muros y canalizaciones. El equipo activo se renueva cada 5 a 7 años; el
cable se espera que dure 15 o 20. Cat6 tiene mejor margen de diafonía que Cat5e y deja camino
para 2.5GBASE-T y 5GBASE-T sin recablear.

Descarté Cat6a para el horizontal por algo muy práctico: su diámetro y su radio de curvatura
mayores encarecen la instalación y ocupan más sección de canalización. Con 48 corridas, ese
sobrecosto no compra ninguna capacidad que estos puestos vayan a usar.

### Cableado troncal

Aquí está la decisión que más me costó, y es la que el enunciado pide justificar de forma
específica.

Elegí **UTP Cat6a** para los ocho enlaces troncales. El razonamiento: cada troncal agrega el
tráfico de un departamento completo. El caso más exigente es la Sala de Capacitación, con 10
hosts que pueden estar descargando material a la vez. Un uplink de 1 Gbps para 10 puestos da una
sobresuscripción de 10:1 — aceptable hoy, pero justo. Cat6a permite llevar ese uplink a 10 Gbps
**sin tocar el cableado**, solo cambiando el equipo activo.

En números: los 147.2 m de troncal en Cat6a en lugar de Cat6 representan alrededor de Q1,150
adicionales sobre un proyecto de más de Q100,000, es decir menos del 2 %. Es la mejora de mayor
retorno de todo el diseño.

**Por qué no fibra óptica.** La evalué seriamente, porque es la respuesta refleja cuando se
habla de troncal. La descarté por distancia: el troncal más largo del edificio mide **22.7 m**.
La fibra se justifica al superar los 100 m del cobre, ante interferencia electromagnética severa,
o cuando hay diferencias de potencial de tierra entre edificios. Ninguna de esas tres condiciones
se da aquí: un solo nivel, una sola acometida eléctrica, distancias cortas.

Además habría exigido un ODF en el MDF, transceptores SFP en los nueve switches y personal con
equipo de fusión. Todo ese costo no compra ninguna capacidad adicional a 22 m, porque Cat6a ya
entrega 10 Gbps a esa distancia. Elegir fibra aquí habría sido sobreingeniería.

---

## 4. Criterios de selección de equipo activo

Dimensioné el patch panel del edificio sobre los 48 puntos de red y, en consecuencia, el switch
principal a 48 puertos. Los switches de departamento son de 24 puertos: el más cargado es el de
Capacitación, con 10 ocupados y 14 libres.

Puse un patch panel de 12 puertos en cada departamento aunque en varios sobren ocho. La razón es
que el patch panel existe para que **el cable rígido de obra no se manipule nunca**: todos los
cambios se hacen con patch cords. Sin él, cada movimiento de un puesto desgasta el conector del
cable que está dentro del muro.

Para la UPS calculé 270 W de carga (nueve switches sin PoE), lo que da 300 VA con factor de
potencia 0.9, y 420 VA aplicando un 40 % de margen de crecimiento. Elegí una unidad de
1500 VA / 900 W y no una de 1000 VA porque a esa capacidad la carga queda en el 30 %, que es el
rango donde una UPS entrega su mejor autonomía.

---

## 5. Retos encontrados al interpretar el plano base

### 5.1 El plano no tiene cuarto de telecomunicaciones

Este fue el primer problema real. Los once espacios del plano están todos asignados a una función
y no hay área ociosa. Crear un cuarto dedicado significaba quitarle metros a algún departamento.

Lo resolví con un **gabinete de pared sobre el Pasillo Secundario**. La decisión no fue por
descarte: el Pasillo Secundario es uno de los dos corredores que cruzan el edificio de lado a
lado, así que la escalerilla troncal nace literalmente en el gabinete, sin necesitar un tramo
dedicado para sacar los cables.

Verifiqué la ubicación con números en lugar de suponerla. El centro geométrico de la planta está
en (12.00, 12.00) m y el MDF quedó a 3.5 m de ahí. Lo que importa es la consecuencia: el troncal
más largo mide 22.7 m de recorrido y el enlace horizontal más largo, 15.0 m — un 17 % del máximo
de 90 m que permite el estándar.

Comparé tres alternativas antes de decidir. Poner el MDF en el Data Center habría subido el
troncal a Recepción de 22.7 m a 30.5 m, y el promedio de los ocho de 12.5 m a 19.1 m. Ponerlo en el Vestíbulo lo dejaba en la zona de acceso público,
que es la peor opción de seguridad física. Un cuarto dedicado quitándole 4 m² a Capacitación era
un poco más central, pero le restaba área justo al departamento con más puntos de red y ganaba
apenas 2 o 3 m.

La debilidad honesta de mi elección es que el corredor es zona de tránsito. La mitigo con
gabinete cerrado con llave montado a 1.80 m, y con 48 puntos en un solo nivel el riesgo residual
me parece aceptable.

### 5.2 El baño rompe la geometría de Recursos Humanos

Recursos Humanos es de 7 × 6 m pero tiene un baño de 2 × 2 m embebido en su esquina inferior
izquierda, lo que deja el departamento con forma de L. Con 8 puntos de red —el segundo más
cargado— la distribución en rejilla no cabía sin invadir el baño.

Reorganicé los equipos en dos filas desplazadas hacia la derecha, dejando libre la franja del
baño, y coloqué dos de las cuatro tomas sobre el muro que da al Pasillo Secundario. Fue un caso
donde la arquitectura obligó a cambiar la disposición, no al revés.

### 5.3 Las puertas condicionan dónde poner las tomas

Cada puerta tiene un arco de barrido que ocupa aproximadamente un metro cuadrado de muro y piso.
Al ubicar las tomas tuve que verificar una por una que ninguna cayera dentro de ese arco: una
toma detrás de una puerta abierta es inaccesible en la práctica. Esto desplazó varias tomas
respecto a mi primera propuesta, sobre todo en Data Center y Backend, cuyas puertas están en el
muro norte justo donde inicialmente había puesto las placas.

### 5.4 Trasladar el plano a la herramienta con precisión

Reproducir los muros, puertas y particiones sobre el diagrama a escala exacta llevó bastante más
trabajo del previsto. Tuve que ajustar varias veces la posición de los equipos para que ninguno
quedara sobre un muro, dentro del baño o fuera de su departamento.

El resultado tiene una ventaja que compensa el esfuerzo: como el diagrama está a escala real,
**todas las distancias del manual salen de las coordenadas y no de mediciones a ojo**. Las
tablas de metraje y el cálculo de bobinas son reproducibles: cualquiera puede verificar cada
longitud sobre el plano.

---

## 6. Verificaciones que hice sobre el diseño

| Verificación | Resultado |
|---|---|
| Ningún enlace horizontal supera 90 m | El más largo mide 15.0 m — 17 % del límite |
| Ningún equipo queda fuera de su departamento | 48 hosts y 9 switches dentro de su espacio |
| Ninguna toma cae dentro del arco de barrido de una puerta | 23 tomas verificadas una por una |
| Cada host tiene un puerto de switch dedicado | 48 puertos, uno por host |
| Los tipos de cable corresponden a los dispositivos conectados | 48 straight-through y 8 crossover |
| Total de equipos según el enunciado | 30 PC + 12 laptops + 6 servidores = 48 |
| Suma de puntos por departamento | 4+4+8+10+4+8+7+3 = 48 |

---

## 7. Lo que haría distinto

**Redundancia del MDF.** Es la debilidad más clara del diseño. Un solo switch principal
concentra todo el edificio. El enunciado pide un único switch principal, así que respeté el
requisito, pero en un proyecto real habría propuesto desde el inicio un segundo switch con
enlaces redundantes hacia Data Center y Backend.

**Certificación desde el presupuesto.** Incluí la certificación de los 56 enlaces como partida,
pero al principio la había omitido. Al investigar TIA/EIA-606 entendí por qué es obligatoria:
las garantías extendidas de fabricante, de 20 a 25 años, exigen documentación de etiquetado y
resultados de certificación como condición de validez. Sin certificar, no hay garantía que
reclamar.

**Margen de cable más holgado.** El cálculo arroja 587 m de Cat6 y dos bobinas dan 610 m: un
margen de apenas 3.8 %. Es demasiado ajustado para obra real, donde un par de errores de corte
obligarían a comprar una bobina de urgencia. Lo dejé señalado en el manual con la recomendación
de adquirir una tercera bobina como reserva.

---

## 8. Conclusión

El aporte principal de este trabajo no está en las decisiones individuales —estrella por
departamento, Cat6 al escritorio, Cat6a en el troncal son elecciones bastante estándar— sino en
haberlas **sostenido con números tomados del propio plano**. La ubicación del MDF se justifica
porque deja el enlace más largo en 15 m; el rechazo de la fibra se justifica porque el troncal
máximo es de 22.7 m; el tamaño de la UPS sale de una carga calculada de 270 W.

Trabajar el plano a escala real desde el inicio fue lo que hizo posible esa trazabilidad, y es
lo que me llevo del ejercicio: en diseño físico, la diferencia entre una propuesta defendible y
una lista de buenas intenciones es si los números se pueden reproducir sobre el plano.
