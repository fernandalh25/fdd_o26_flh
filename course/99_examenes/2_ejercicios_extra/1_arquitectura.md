---
id: ejercicios-arquitectura
title: "Ejercicios extra de arquitectura y SO"
nav_title: "Arquitectura y SO"
summary: "Ejercicios con la forma del parcial 1, cada uno con una pista y su respuesta plegadas: jerarquía de memoria, localidad, CPU contra GPU, FLOPS y tiempos, el techo de la memoria, GHz, ISA y el sistema operativo."
status: ready
estimated_time: 40m
tags: [ejercicios, arquitectura, memoria, cpu, gpu, flops, sistemas-operativos]
---

# Ejercicios extra de arquitectura y SO

**[PDF sin respuestas, para imprimir](../_assets/practica-arquitectura.pdf)** · unos 40 minutos · sin apuntes

Miden lo mismo que el [[parcial-arquitectura|parcial 1]] con casos nuevos. Debajo de cada pregunta hay dos cosas plegadas:

- **la pista**: ábrela sólo si llevas un rato atorado;
- **la respuesta**, con su explicación.

Varias preguntas piden una cuenta. Hazla en papel: son órdenes de magnitud, y lo que importa es el razonamiento.

## Parte 1 · Memoria

::: problem {#xa-1 title="1 · Ordena por velocidad"}
Ordena del más rápido al más lento: SSD, caché L2, registro, RAM, red, caché L1, caché L3. ¿Qué pasa con la capacidad en ese mismo orden?
:::

::: hint {of="xa-1"}
Piensa en la distancia al lugar donde se hace la cuenta. Lo que está dentro del núcleo es lo más rápido.
:::

::: answer {of="xa-1"}
**Registro → L1 → L2 → L3 → RAM → SSD → red.**

La capacidad va **al revés**: los registros guardan bytes, y la red, casi lo que quieras. Al acercarse al cómputo, cada nivel es más rápido, más pequeño y más caro por byte. Por eso existe la jerarquía: no hay una memoria rápida, grande y barata a la vez.
:::

::: problem {#xa-2 title="2 · No cupo"}
Tu laptop tiene 16 GB de RAM. Procesar una tabla de 2 GB tarda 10 segundos. Una tabla de 40 GB, con el mismo procesamiento, tarda mucho más que 20 veces eso. ¿Por qué no escala en proporción?
:::

::: hint {of="xa-2"}
¿Dónde quedan los 40 GB si la RAM tiene 16? ¿Qué nivel de la jerarquía toca entonces cada acceso?
:::

::: answer {of="xa-2"}
**Los 40 GB no caben en 16 GB de RAM: una parte vive en el SSD, y cada acceso a esa parte es unas mil veces más lento.**

| Tabla | ¿Cabe en 16 GB? | Dónde ocurre cada acceso |
|---|---|---|
| 2 GB | Sí | RAM |
| 40 GB | **No** | RAM + **SSD** |

Con 40 GB, el sistema operativo manda pedazos al SSD y los vuelve a traer cuando se necesitan.

La diferencia entre esos dos niveles, con las latencias orientativas de [[memoria-y-datos|Memoria y movimiento de datos]]:

| Nivel | Latencia orientativa |
|---|---|
| RAM | 60–150 ns |
| SSD NVMe | 50–300 µs |

50 µs ÷ 60 ns ≈ 830 y 300 µs ÷ 150 ns = 2,000: del orden de **mil veces**. Ya no haces 20 veces el mismo trabajo: haces el trabajo **un nivel más abajo** en la jerarquía.

Es el síntoma «no cabe» del curso: el límite es la **capacidad**. Arreglos posibles: procesar por pedazos que quepan, usar sólo las columnas necesarias, o una máquina con más RAM.
:::

::: problem {#xa-3 title="3 · En orden o al azar"}
Dos programas leen los mismos 1,000 millones de números de un arreglo en RAM. Uno los recorre **en orden** y el otro **saltando al azar**. Leen la misma cantidad de datos. ¿Cuál termina antes, y por qué?
:::

::: hint {of="xa-3"}
Cuando la CPU trae un dato de la RAM, no trae sólo ese: trae un bloque con sus vecinos. ¿A cuál de los dos programas le sirven los vecinos?
:::

::: answer {of="xa-3"}
**El que va en orden, por mucho.**

Cada viaje a la RAM trae a la caché un **bloque** de números contiguos, no uno solo. Para el ejemplo, supón que un bloque trae 8 números:

| Programa | Posición que lee | ¿Ya está en caché? | Paga |
|---|---|---|---|
| En orden | 0 | No: trae el bloque 0–7 | latencia de RAM |
| En orden | 1, 2, …, 7 | **Sí**: llegaron con el bloque | casi nada |
| Al azar | 512,334,019 | No: trae su bloque | latencia de RAM |
| Al azar | 88,120 | No: es otro bloque | **latencia de RAM otra vez** |

En orden, 7 de cada 8 lecturas ya están en la caché. Además, el procesador ve el patrón y trae el siguiente bloque antes de que lo pidas. Al azar, casi cada lectura cae en un bloque nuevo: usas un número y desperdicias los otros siete.

Esto es la **localidad**. Un recorrido en orden aprovecha el **ancho de banda**; uno al azar queda dominado por la **latencia**. Misma cantidad de números, métrica distinta.
:::

**Repasa:** [[memoria-y-datos|Memoria y movimiento de datos]].

## Parte 2 · CPU, GPU y FLOPS

::: problem {#xa-4 title="4 · ¿CPU o GPU?"}
Para cada tarea, elige CPU o GPU y di por qué en una frase:

a) Compilar un programa.

b) Multiplicar dos matrices de 10,000 × 10,000.

c) Un servidor web que atiende solicitudes pequeñas, cada una con su lógica.

d) Aplicar el mismo filtro a un millón de imágenes.

e) Leer un JSON con estructura irregular y validar cada campo.
:::

::: hint {of="xa-4"}
Pregúntate en cada caso: ¿es la misma operación sobre muchos datos independientes, o son decisiones distintas paso a paso?
:::

::: answer {of="xa-4"}
| Tarea | Elige | Por qué |
|---|---|---|
| a) Compilar | **CPU** | Lleno de decisiones y ramas; cada paso depende del anterior |
| b) Matrices | **GPU** | Millones de multiplicaciones iguales e independientes |
| c) Servidor web | **CPU** | Solicitudes pequeñas con lógica distinta; importa la latencia de cada una |
| d) Filtro a imágenes | **GPU** | La misma operación sobre muchísimos píxeles |
| e) JSON irregular | **CPU** | Cada campo se valida distinto: control irregular |

En b) y d), la GPU gana si el volumen justifica copiar los datos a su memoria. Con pocas imágenes, la copia puede costar más que el cálculo (ejercicio 7).
:::

::: problem {#xa-5 title="5 · ¿Cuánto tarda el entrenamiento?"}
Entrenar un modelo requiere **6 × 10¹⁸ FLOP**. Tu GPU anuncia **300 TFLOPS**, pero en la práctica tu código aprovecha sólo el **40 %**. ¿Cuánto tarda, en horas?
:::

::: hint {of="xa-5"}
FLOP es trabajo y FLOPS es ritmo: tiempo = trabajo / ritmo. ¿Cuánto vale «tera»? Aplica el 40 % al ritmo.
:::

::: answer {of="xa-5"}
**Unas 13.9 horas.**

| Paso | Cuenta | Resultado |
|---|---|---|
| Ritmo anunciado | 300 TFLOPS = 300 × 10¹² FLOP/s | 3 × 10¹⁴ FLOP/s |
| Ritmo real (40 %) | 3 × 10¹⁴ FLOP/s × 0.4 | **1.2 × 10¹⁴ FLOP/s** |
| Tiempo en segundos | 6 × 10¹⁸ FLOP ÷ (1.2 × 10¹⁴ FLOP/s) | **5 × 10⁴ s** |
| Tiempo en horas | 5 × 10⁴ s ÷ 3,600 s/h | **≈ 13.9 h** |

El número anunciado es un **pico**. El 40 % es realista: el código espera datos, sincroniza y no usa todas las unidades todo el tiempo.

Ojo también con la **precisión**, que es cuántos bits ocupa cada número: FP32 usa 32 bits (4 bytes) y BF16 usa 16 (2 bytes). El pico de una GPU suele anunciarse en una precisión baja como BF16; si tu código calcula en FP32, el mismo chip hace bastante menos FLOP por segundo y el entrenamiento tarda más. Lo ves en [[paralelismo-performance-energia|Paralelismo, performance y energía]].
:::

::: problem {#xa-6 title="6 · El techo de la memoria"}
Un chip hace **2 TFLOPS** y su memoria entrega **100 GB/s**. Calculas `C[i] = A[i] + B[i]` con números FP32, de 4 bytes. Por cada suma lees A y B y escribes C.

a) ¿Cuántos FLOP haces por byte movido?

b) ¿Cuántos GFLOPS logras como máximo?

c) ¿Ayuda cambiar a un chip con el doble de FLOPS y la misma memoria?
:::

::: hint {of="xa-6"}
Una suma es 1 FLOP. ¿Cuántos bytes mueves por suma? Si cada byte permite tantos FLOP y llegan 100 GB por segundo, ¿cuántos FLOP por segundo alcanzas?
:::

::: answer {of="xa-6"}
**a) 1/12 ≈ 0.083 FLOP/byte · b) ≈ 8.3 GFLOPS · c) No.**

La **intensidad aritmética** es cuántos FLOP haces por cada byte que mueves: FLOP ÷ bytes. Es la cuenta del Roofline de [[paralelismo-performance-energia|Paralelismo, performance y energía]], con los mismos números.

| Paso | Cuenta | Resultado |
|---|---|---|
| Bytes por suma | 4 (leer A) + 4 (leer B) + 4 (escribir C) | 12 bytes |
| FLOP por suma | una suma | 1 FLOP |
| a) Intensidad | 1 FLOP ÷ 12 bytes | **1/12 ≈ 0.083 FLOP/byte** |
| Techo de memoria | 100 GB/s × 1/12 FLOP/byte | 8.3 GFLOPS |
| Techo de cómputo | 2 TFLOPS | 2,000 GFLOPS |
| b) Manda el menor | menor de 8.3 y 2,000 | **8.3 GFLOPS** |

0.083 es 1/12 redondeado. Los bytes se cancelan: (bytes/s) × (FLOP/byte) = FLOP/s.

8.3 GFLOPS es menos del 0.5 % de los 2,000 GFLOPS: las unidades de cálculo pasan casi todo el tiempo esperando datos.

**c)** Cambia un techo a la vez y vuelve a tomar el menor:

| Chip | Techo de memoria | Techo de cómputo | Manda |
|---|---|---|---|
| Original | 8.3 GFLOPS | 2,000 GFLOPS | 8.3 GFLOPS |
| Doble de FLOPS | 8.3 GFLOPS | **4,000 GFLOPS** | 8.3 GFLOPS |
| Doble de ancho de banda (200 GB/s) | **16.7 GFLOPS** | 2,000 GFLOPS | **16.7 GFLOPS** |

El doble de FLOPS no cambia nada: el límite es la memoria. Lo que ayuda es más ancho de banda.

**La regla:** el rendimiento es el **menor** de dos techos, el de cómputo y el de ancho de banda × intensidad.
:::

::: problem {#xa-7 title="7 · ¿Vale la pena la copia?"}
Tienes 1 GB de datos en RAM. La GPU los procesa en 5 ms, y la CPU en 200 ms. Copiar a la GPU viaja a unos 25 GB/s, y el resultado de vuelta es pequeño.

a) ¿Quién termina antes?

b) ¿Y si la CPU tardara 30 ms?
:::

::: hint {of="xa-7"}
El tiempo de la GPU no es sólo su cálculo: suma lo que tarda el viaje de los datos hasta ella.
:::

::: answer {of="xa-7"}
**a) GPU: 45 ms contra 200 ms · b) CPU: 30 ms contra 45 ms.**

El tiempo de la GPU:

| Paso | Cuenta | Tiempo |
|---|---|---|
| Copiar 1 GB a la GPU | 1 GB ÷ 25 GB/s = 0.04 s | 40 ms |
| Calcular en la GPU | dato del enunciado | 5 ms |
| Traer el resultado | es pequeño | ≈ 0 ms |
| **Total GPU** | 40 + 5 | **45 ms** |

La comparación:

| Caso | CPU | GPU (copia + cálculo) | Gana |
|---|---|---|---|
| a) | 200 ms | 45 ms | **GPU** |
| b) | 30 ms | 45 ms | **CPU** |

En b), la copia sola (40 ms) ya tarda más que todo el trabajo en CPU (30 ms).

Por eso un trabajo corto rara vez conviene en la GPU, y por eso conviene dejar los datos en la GPU entre un paso y el siguiente, en vez de copiarlos ida y vuelta.
:::

::: problem {#xa-8 title="8 · GHz contra trabajo"}
El chip X corre a **5 GHz** y termina en promedio **2 instrucciones por ciclo**. El chip Y corre a **3.5 GHz** y termina **4 instrucciones por ciclo**. Para un programa de un solo hilo que no espera a la memoria, ¿cuál es más rápido?
:::

::: hint {of="xa-8"}
Instrucciones por segundo = ciclos por segundo × instrucciones por ciclo.
:::

::: answer {of="xa-8"}
**Y, que es 40 % más rápido aunque tiene menos GHz.**

| Chip | Ciclos por segundo | Instrucciones por ciclo | Instrucciones por segundo |
|---|---|---|---|
| X | 5 GHz = 5 × 10⁹ | 2 | 5 × 10⁹ × 2 = 10 × 10⁹ |
| Y | 3.5 GHz = 3.5 × 10⁹ | 4 | 3.5 × 10⁹ × 4 = **14 × 10⁹** |

(14 × 10⁹) ÷ (10 × 10⁹) = 1.4: Y termina 40 % más instrucciones por segundo. Los GHz miden el ritmo del reloj, no el trabajo que sale de cada tick; eso depende de la microarquitectura. Y esto vale sólo con la condición del enunciado: si el programa espera a la memoria, ninguno de los dos números manda.
:::

**Repasa:** [[paralelismo-performance-energia|Paralelismo, performance y energía]] y [[compute-instrucciones-cpu|Compute, instrucciones y CPU]].

## Parte 3 · ISA y sistema operativo

::: problem {#xa-9 title="9 · El binario que no corre"}
Compilas un programa en C en tu laptop x86-64 y copias el ejecutable a una Raspberry Pi, que es ARM. ¿Corre? ¿Y un script de Python copiado igual?
:::

::: hint {of="xa-9"}
¿En qué está escrito un ejecutable compilado? ¿Y quién lee un script de Python?
:::

::: answer {of="xa-9"}
**El binario no corre; el script sí, si la Pi tiene Python instalado.**

- **El binario de C no corre.** Está escrito en instrucciones de la **ISA x86-64**, y la Pi entiende las de **ARM**: son contratos distintos. Hay que recompilarlo para ARM. El comando `file` dice para qué ISA está compilado un ejecutable:

```text
$ gcc hola.c -o hola
$ file hola
hola: ELF 64-bit LSB pie executable, x86-64, ...
```

  Copiado a la Pi y ejecutado, bash lo rechaza:

```text
$ ./hola
bash: ./hola: cannot execute binary file: Exec format error
```

- **El script de Python sí corre**, si la Pi tiene Python instalado. El script no son instrucciones de máquina: lo lee el **intérprete**, y el intérprete de la Pi ya está compilado para ARM.

Matiz: si el script usa bibliotecas con partes compiladas, como numpy, esas partes deben existir en versión ARM. Por eso a veces `pip install` funciona en una máquina y en otra no.
:::

::: problem {#xa-10 title="10 · ¿Quién habla con el disco?"}
En Python escribes `open("datos/ventas.csv").read()`. ¿Tu programa le habla al disco directamente? Nombra dos cosas que hace el sistema operativo en ese momento.
:::

::: hint {of="xa-10"}
Recuerda la definición del curso: el sistema operativo es el intermediario. ¿Qué traduce, qué controla y qué reparte?
:::

::: answer {of="xa-10"}
**No. Tu programa le pide al sistema operativo que lea el archivo, y el sistema operativo hace el resto.**

Lo que pasa con esa línea, en orden:

| Paso | Quién | Qué hace |
|---|---|---|
| 1 | Python | Pide al sistema operativo abrir y leer `datos/ventas.csv` |
| 2 | Sistema operativo | **Traduce el nombre** a los bloques del disco donde viven esos bytes: es el **sistema de archivos** |
| 3 | Sistema operativo | **Revisa los permisos**: si tu usuario puede leer ese archivo |
| 4 | Sistema operativo | Si hace falta, **controla el disco** a través de su driver |
| 5 | Sistema operativo | **Pone los bytes en la memoria** de tu programa |
| mientras | Sistema operativo | **Reparte la CPU**: mientras el disco responde, deja correr a otros programas |

La pregunta pide dos: **cualesquiera dos de las filas del sistema operativo, bien explicadas, bastan.** Las demás son extra.

El paso 3 se prueba quitándole los permisos al archivo (como usuario normal, no root). Python no decide nada: recibe el rechazo del sistema operativo (`Errno 13`).

```text
$ chmod 000 datos/ventas.csv
$ python3 -c 'open("datos/ventas.csv").read()'
Traceback (most recent call last):
...
PermissionError: [Errno 13] Permission denied: 'datos/ventas.csv'
```

Por esta mediación, la misma línea de Python puede comportarse distinto en Windows y en Linux: cambian las rutas, los permisos y el sistema de archivos.
:::

::: problem {#xa-11 title="11 · ¿Dónde está el cuello de botella?"}
Un ETL nocturno lee **2 TB** de CSV desde un SSD que entrega unos **2 GB/s**, filtra filas con una condición simple y escribe un resultado pequeño. Alguien propone comprar una GPU para acelerarlo. ¿Ayudaría?
:::

::: hint {of="xa-11"}
Calcula cuánto tarda sólo en **leer** los 2 TB. Luego pregúntate cuánto cálculo hay por cada byte leído.
:::

::: answer {of="xa-11"}
**Casi seguro que no: el cuello de botella es leer del SSD, no calcular.**

| Paso | Cuenta | Resultado |
|---|---|---|
| Leer 2 TB a 2 GB/s | 2,000 GB ÷ 2 GB/s | **1,000 s ≈ 17 min** |
| Cálculo por byte leído | una condición simple por fila | muy poco |
| Con GPU | los datos siguen llegando a 2 GB/s, y además hay que copiarlos a la GPU | **≥ 1,000 s** |

Ningún procesador termina antes de que lleguen los datos, y los datos tardan 1,000 s en salir del SSD. El trabajo está limitado por el **almacenamiento**: una GPU esperaría datos igual que la CPU.

Lo que sí ayuda, porque ataca la lectura:

- un formato **columnar** como Parquet, para leer sólo las columnas necesarias;
- comprimir, para leer menos bytes;
- un almacenamiento más rápido;
- repartir la lectura entre varias máquinas.

Antes de comprar hardware, hay que preguntar qué recurso se agota primero: capacidad, latencia, ancho de banda o cálculo.
:::

**Repasa:** [[compute-instrucciones-cpu|Compute, instrucciones y CPU]], [[software-libre-y-sistemas-operativos|Software libre y sistemas operativos]] y [[ia-escala-decision|IA, escala y selección de hardware]].
