---
id: ejercicios-docker
title: "Ejercicios extra de Docker"
nav_title: "Docker"
summary: "Ejercicios nuevos con la forma del parcial de Docker, cada uno con su respuesta explicada debajo: un Dockerfile con .dockerignore y caché, una secuencia de volúmenes de doce filas y cinco errores reales."
status: ready
estimated_time: 40m
tags: [ejercicios, docker, dockerfile, dockerignore, volumenes, cache, errores]
---

# Ejercicios extra de Docker

**[PDF sin respuestas, para imprimir](../_assets/practica-docker.pdf)** · unos 40 minutos · sin apuntes

Miden lo mismo que el parcial, con escenarios nuevos. Debajo de cada pregunta hay una **pista** y la **respuesta**, plegadas. Abre la pista sólo si llevas un rato atorado. Todas las respuestas se comprobaron corriendo los comandos con Docker 29.6: si quieres, córrelos tú también.

## Parte 1 · Lee el Dockerfile

`encuestas/` es un repo de Python. El programa `limpiar.py` usa `reglas.py`, lee `crudos/respuestas.csv` y escribe `salida/limpio.csv`. Las dos rutas son relativas a la carpeta donde corre.

```text
encuestas/        ← estás aquí
├── .dockerignore
├── Dockerfile
├── README.md
├── requirements.txt
├── limpiar.py
├── reglas.py
├── notas/
│   └── ideas.md
├── crudos/
│   └── respuestas.csv
└── salida/
```

El `.dockerignore`:

```text
crudos/
notas/
salida/
```

El `Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "limpiar.py"]
```

La imagen ya se construyó una vez con `docker build -t encuestas:1 .`.

::: problem {#xd-1a title="a · ¿Para qué se copia requirements.txt antes?"}
El `COPY . .` de la línea 5 ya copia `requirements.txt`. ¿Para qué se copia antes, por separado, en la línea 3?
:::

::: hint {of="xd-1a"}
Docker reusa una capa mientras no cambie lo que esa capa lee. ¿De qué depende el `pip install` si `requirements.txt` va solo?
:::

::: answer {of="xd-1a"}
**Por la caché.** Docker reusa una capa mientras no cambie nada de lo que esa capa lee. Con `requirements.txt` solo, el `pip install` depende únicamente de ese archivo: si cambias tu código, la instalación, que es lo lento, sale de la caché. Con un solo `COPY . .` antes del `pip install`, cualquier cambio reinstalaría todo.
:::

::: problem {#xd-1b title="b · ¿Qué queda dentro de /app?"}
¿Qué archivos y carpetas quedan dentro de `/app` en la imagen?
:::

::: hint {of="xd-1b"}
`COPY . .` copia todo el contexto menos lo que excluye el `.dockerignore`. ¿Alguien excluyó el propio Dockerfile?
:::

::: answer {of="xd-1b"}
**`.dockerignore`, `Dockerfile`, `README.md`, `limpiar.py`, `reglas.py` y `requirements.txt`** (comprobado con `ls -A /app`).

`COPY . .` copia todo el contexto **menos** lo que excluye el `.dockerignore`. No entran `crudos/`, `notas/` ni `salida/`. Sí entran el Dockerfile y el propio `.dockerignore`, porque nadie los excluyó.
:::

::: problem {#xd-1c title="c · Córrelo con las dos carpetas montadas"}
Escribe el comando que corre `encuestas:1` en un contenedor que se borre al terminar, con tu carpeta `crudos/` montada en `/app/crudos` y tu carpeta `salida/` montada en `/app/salida`.
:::

::: hint {of="xd-1c"}
Un `-v` por carpeta, los dos antes del nombre de la imagen.
:::

::: answer {of="xd-1c"}
```bash
docker run --rm -v "$(pwd)/crudos":/app/crudos -v "$(pwd)/salida":/app/salida encuestas:1
```

Va un `-v` por carpeta, los dos **antes** de la imagen. Al terminar, `salida/limpio.csv` está en tu disco.

**Ojo:** `-v crudos:/app/crudos`, sin `./`, crea un named volume vacío; no monta tu carpeta.
:::

::: problem {#xd-1d title="d · ¿Y sin montar nada?"}
Alguien corre `docker run --rm encuestas:1`, sin montar nada. ¿Qué pasa y por qué?
:::

::: hint {of="xd-1d"}
¿Está `crudos/` dentro de la imagen?
:::

::: answer {of="xd-1d"}
Truena con `FileNotFoundError: … 'crudos/respuestas.csv'`. El `.dockerignore` dejó `crudos/` fuera de la imagen a propósito: los datos se montan al correr, no se meten a la imagen.
:::

::: problem {#xd-1e title="e · ¿Qué pasos se rehacen?"}
Después del primer build cambias **un solo** archivo y vuelves a construir. En cada caso, ¿qué pasos se rehacen y cuáles salen de la caché?

1. Agregas una línea a `notas/ideas.md`.
2. Cambias `reglas.py`.
3. Agregas un paquete a `requirements.txt`.
4. Corriges un typo en `README.md`.
:::

::: hint {of="xd-1e"}
Para cada archivo, pregúntate qué `COPY` lo lee, y si ese archivo existe para el build.
:::

::: answer {of="xd-1e"}
Cuando una capa se rehace, todas las que siguen también.

| Cambio | `COPY requirements.txt .` | `RUN pip install` | `COPY . .` |
|---|---|---|---|
| 1. `notas/ideas.md` | caché | caché | caché |
| 2. `reglas.py` | caché | caché | **se rehace** |
| 3. `requirements.txt` | **se rehace** | **se rehace** | **se rehace** |
| 4. `README.md` | caché | caché | **se rehace** |

El caso 1 es el que sorprende: `notas/` está en el `.dockerignore`, así que para el build ese archivo no existe y nada cambió. Los cuatro casos se comprobaron con `--progress=plain`.
:::

::: problem {#xd-1f title="f · ¿Y si se olvida salida/ en el .dockerignore?"}
Alguien borra la línea `salida/` del `.dockerignore`. Corre el programa con la salida montada, como en c), y vuelve a construir. ¿Qué dos cosas cambian?
:::

::: hint {of="xd-1f"}
¿Qué escribe el programa en `salida/`, y qué copia `COPY . .` si ya no la excluye?
:::

::: answer {of="xd-1f"}
**Dos cosas: `limpio.csv` entra a la imagen, y el `COPY . .` se rehace cada vez que una corrida deja un `limpio.csv` distinto.**

| Paso | `salida/` en tu disco | `COPY . .` | `/app/salida` en la imagen |
|---|---|---|---|
| corres con `salida/` montada | **`limpio.csv`** | | · |
| `docker build` | `limpio.csv` | **se rehace** | **`limpio.csv`** |
| corres con datos nuevos | **`limpio.csv` distinto** | | `limpio.csv` viejo |
| `docker build` | `limpio.csv` distinto | **se rehace** | **`limpio.csv` nuevo** |

`COPY . .` ya no excluye `salida/`, así que copia lo que el programa escribió en tu disco. Docker compara el **contenido** de los archivos, no su fecha: si la corrida reescribe el mismo `limpio.csv`, el `COPY` sale de la caché.

```text
$ docker run --rm encuestas:1 ls -A /app/salida
limpio.csv

# corres otra vez con los mismos datos y construyes
#9 [5/5] COPY . .
#9 CACHED

# corres con datos nuevos y construyes
#9 [5/5] COPY . .
#9 DONE 0.0s
```

Por eso las carpetas de datos y de resultados van en el `.dockerignore`: los resultados no deben viajar dentro de la imagen, ni decidir cuándo se rehace una capa.
:::

::: problem {#xd-1g title="g · ¿Corre limpiar.py?"}
`docker run --rm encuestas:1 python reglas.py`, ¿corre `limpiar.py`? ¿Por qué?
:::

::: hint {of="xd-1g"}
¿Qué pasa con el `CMD` cuando escribes algo después del nombre de la imagen?
:::

::: answer {of="xd-1g"}
No. Lo que escribes después del nombre de la imagen **reemplaza** al `CMD`. Corre `python reglas.py`, que sólo define cosas y termina sin imprimir nada.
:::

**Repasa:** [[el-dockerfile-por-dentro]], [[capas-y-cache]] y [[rutas-en-docker]].

## Parte 2 · ¿Qué cambió?

En `lab/` hay sólo este Dockerfile y `saludo.txt`, cuyo texto es la palabra **hola**:

```dockerfile
FROM alpine:3.20
WORKDIR /s
COPY saludo.txt .
CMD ["cat", "saludo.txt"]
```

No existen imágenes, contenedores, volúmenes ni caché previos. Los comandos se corren **en orden**, todos desde `lab/`, y ningún `sleep 600` alcanza a terminar. Cada fila parte de lo que dejó la anterior.

En cada fila, pregúntate: ¿este contenedor lee la imagen, tu carpeta, un volumen o su propia capa de escritura?

::: problem {#xd-2-1 title="Fila 1 · ¿Qué imprime el último exec?"}
```bash
docker build -t saludo:1 .
docker run -d --name pa saludo:1 sleep 600
docker exec pa sh -c 'echo adios > saludo.txt'
docker stop pa
docker start pa
docker exec pa cat saludo.txt
```
:::

::: hint {of="xd-2-1"}
Estado al empezar:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| inicio | `hola` | · | · | · |  |

¿Qué borra `docker stop`? ¿Qué borra `docker rm`?
:::

::: answer {of="xd-2-1"}
**`adios`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| inicio | `hola` | · | · | · |  |
| `build` | `hola` | **`hola`** | · | · |  |
| `run -d --name pa` | `hola` | `hola` | **pa: corriendo, capa vacía** | · |  |
| `exec pa … adios` | `hola` | `hola` | pa: corriendo, **capa `adios`** | · |  |
| `stop pa` | `hola` | `hola` | pa: **detenido**, capa `adios` | · |  |
| `start pa` | `hola` | `hola` | pa: **corriendo**, capa `adios` | · |  |
| `exec pa cat` | `hola` | `hola` | pa: corriendo, capa `adios` | · | **`adios`** |

`docker stop` detiene el proceso, pero **no borra** el contenedor ni su capa de escritura. `docker start` lo arranca con la misma capa. Sólo `docker rm` la borra.
:::

::: problem {#xd-2-2 title="Fila 2 · ¿Qué imprime?"}
```bash
docker run --rm saludo:1
```
:::

::: hint {of="xd-2-2"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `exec pa cat` | `hola` | `saludo:1` = el build de la fila 1 | pa: corriendo, capa `adios` | · | **`adios`** |

Un contenedor nuevo, sin montaje, ¿de dónde lee?
:::

::: answer {of="xd-2-2"}
**`hola`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm saludo:1` | `hola` | `hola` | pa: corriendo, capa `adios` | · | **`hola`** |

`docker run` crea un contenedor **nuevo**, con capa vacía y sin montaje: lee el `/s` de la imagen. Al salir, `--rm` lo borra. El `adios` está en la capa de pa, y esa capa es sólo de pa.
:::

::: problem {#xd-2-3 title="Fila 3 · ¿Qué imprime el último run?"}
```bash
docker run -d --name pb -v caja:/s saludo:1 sleep 600
docker exec pb sh -c 'echo luna > saludo.txt'
docker run --rm -v caja:/s saludo:1
```
:::

::: hint {of="xd-2-3"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm saludo:1` | `hola` | `hola` | pa: corriendo, capa `adios` | · | **`hola`** |

¿Con qué llena Docker un volumen nuevo y vacío? ¿Qué ve otro contenedor que monta el mismo volumen?
:::

::: answer {of="xd-2-3"}
**`luna`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run -d --name pb -v caja:/s` | `hola` | `hola` | pa: corriendo, capa `adios`; **pb: corriendo, monta `caja`** | **`hola`** (nuevo y vacío: copia de la imagen) |  |
| `exec pb … luna` | `hola` | `hola` | pa: corriendo, capa `adios`; pb: corriendo, monta `caja` | **`luna`** |  |
| `run --rm -v caja:/s` | `hola` | `hola` | pa: corriendo, capa `adios`; pb: corriendo, monta `caja` | `luna` | **`luna`** |

`caja` no existía: Docker lo creó vacío y le copió el `/s` de la imagen (`hola`). Como `WORKDIR` es `/s`, el `echo` de pb escribe en el volumen, no en su capa. El último `run` monta **el mismo** volumen, así que lee `luna`.
:::

::: problem {#xd-2-4 title="Fila 4 · ¿Qué imprime el run?"}
```bash
echo sol > saludo.txt
docker build -t saludo:1 .
docker run --rm -v caja:/s saludo:1
```
:::

::: hint {of="xd-2-4"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm -v caja:/s` | `hola` | `hola` | pa: corriendo, capa `adios`; pb: corriendo, monta `caja` | `luna` | **`luna`** |

¿Docker vuelve a llenar un volumen que ya tiene algo?
:::

::: answer {of="xd-2-4"}
**`luna`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `echo sol > saludo.txt` | **`sol`** | `hola` | pa: corriendo, capa `adios`; pb: corriendo, monta `caja` | `luna` |  |
| `build` | `sol` | **`sol`** | **pa: capa `adios`, imagen `hola`**; pb: corriendo, monta `caja` | `luna` |  |
| `run --rm -v caja:/s` | `sol` | `sol` | pa: capa `adios`, imagen `hola`; pb: corriendo, monta `caja` | `luna` | **`luna`** |

La regla: **Docker copia el contenido de la imagen a un volumen con nombre sólo si el volumen está vacío al montarlo.** `caja` ya tiene `luna`, así que Docker no copia nada. El volumen tapa lo que la imagen tiene en `/s`, y la imagen nueva (`sol`) no se ve.

```text
$ docker run --rm -v caja:/s saludo:1
luna
```
:::

::: problem {#xd-2-5 title="Fila 5 · ¿Qué imprime?"}
```bash
docker run --rm saludo:1
```
:::

::: hint {of="xd-2-5"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm -v caja:/s` | `sol` | `saludo:1` = el build de la fila 4 | pa: capa `adios`, imagen `hola`; pb: corriendo, monta `caja` | `luna` | **`luna`** |

Sin montaje: ¿qué dice la imagen después del último build?
:::

::: answer {of="xd-2-5"}
**`sol`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm saludo:1` | `sol` | `sol` | pa: capa `adios`, imagen `hola`; pb: corriendo, monta `caja` | `luna` | **`sol`** |

Sin montaje, el contenedor nuevo lee la imagen, y `saludo:1` es la que construiste en la fila 4. La misma imagen con `caja` montado da otra cosa, porque el volumen la tapa:

```text
$ docker run --rm -v caja:/s saludo:1
luna
$ docker run --rm saludo:1
sol
```
:::

::: problem {#xd-2-6 title="Fila 6 · ¿Qué imprime?"}
```bash
docker exec pb cat saludo.txt
```
:::

::: hint {of="xd-2-6"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm saludo:1` | `sol` | `sol` | pa: capa `adios`, imagen `hola`; pb: corriendo, monta `caja` | `luna` | **`sol`** |

¿Qué monta pb?
:::

::: answer {of="xd-2-6"}
**`luna`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `exec pb cat` | `sol` | `sol` | pa: capa `adios`, imagen `hola`; pb: corriendo, monta `caja` | `luna` | **`luna`** |

pb monta `caja` en `/s`, y `caja` dice `luna`. Lo que diga la imagen no importa: el volumen la tapa.
:::

::: problem {#xd-2-7 title="Fila 7 · ¿Qué imprime el run?"}
```bash
docker rm -f pb
docker volume rm caja
docker run --rm -v caja:/s saludo:1
```
:::

::: hint {of="xd-2-7"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `exec pb cat` | `sol` | `saludo:1` = el build de la fila 4 | pa: capa `adios`, imagen `hola`; pb: corriendo, monta `caja` | `luna` | **`luna`** |

Si borras el volumen, el `caja` siguiente es nuevo. ¿Con qué se llena uno nuevo?
:::

::: answer {of="xd-2-7"}
**`sol`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `rm -f pb` | `sol` | `sol` | **pb borrado**; pa: capa `adios`, imagen `hola` | `luna` |  |
| `volume rm caja` | `sol` | `sol` | pa: capa `adios`, imagen `hola` | **·** |  |
| `run --rm -v caja:/s` | `sol` | `sol` | pa: capa `adios`, imagen `hola` | **`sol`** (nuevo y vacío: copia de la imagen) | **`sol`** |

Primero se borra pb, porque Docker no deja borrar un volumen que un contenedor usa. Al borrar el volumen, `luna` desaparece. El `caja` del último `run` es **nuevo** y vacío, así que Docker lo llena con la imagen actual: `sol`. `--rm` borra el contenedor, pero no un volumen con nombre: `caja` sigue ahí, con `sol`.

```text
$ docker volume ls -q
caja
```
:::

::: problem {#xd-2-8 title="Fila 8 · ¿Qué imprime?"}
```bash
docker exec pa cat saludo.txt
```
:::

::: hint {of="xd-2-8"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm -v caja:/s` | `sol` | `sol` | pa: capa `adios`, imagen `hola` | **`sol`** (nuevo y vacío: copia de la imagen) | **`sol`** |

¿Cambia un contenedor que ya existe cuando reconstruyes su imagen?
:::

::: answer {of="xd-2-8"}
**`adios`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `exec pa cat` | `sol` | `sol` | pa: capa `adios`, imagen `hola` | `sol` | **`adios`** |

Un contenedor queda atado a la imagen con la que se creó. pa nació en la fila 1 con la imagen `hola`, y su capa dice `adios`. El build de la fila 4 movió el nombre `saludo:1` a una imagen nueva, pero pa sigue con la vieja.

Lo prueban los dos identificadores, que no coinciden:

```text
$ docker inspect -f '{{.Image}}' pa | cut -c1-19
sha256:2c54278b3d3f
$ docker image inspect -f '{{.Id}}' saludo:1 | cut -c1-19
sha256:03c95815ec57
```
:::

::: problem {#xd-2-9 title="Fila 9 · ¿Qué imprime cada run? (dos casillas)"}
```bash
echo nube > saludo.txt
docker run --rm -v "$(pwd)":/s saludo:1
docker run --rm saludo:1
```
:::

::: hint {of="xd-2-9"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `exec pa cat` | `sol` | `saludo:1` = el build de la fila 4 | pa: capa `adios`, imagen `hola` | `sol` | **`adios`** |

Con tu carpeta montada en `/s`, ¿qué lee? ¿Y sin montar nada?
:::

::: answer {of="xd-2-9"}
- **Primero: `nube`.**
- **Segundo: `sol`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `echo nube > saludo.txt` | **`nube`** | `sol` | pa: capa `adios`, imagen `hola` | `sol` |  |
| `run --rm -v "$(pwd)":/s` | `nube` | `sol` | pa: capa `adios`, imagen `hola` | `sol` | **`nube`** |
| `run --rm saludo:1` | `nube` | `sol` | pa: capa `adios`, imagen `hola` | `sol` | **`sol`** |

El primero monta tu carpeta en `/s`: lee tu disco. El segundo no monta nada: lee la imagen, y no ha habido build desde la fila 4. Editar tu carpeta no cambia la imagen.
:::

::: problem {#xd-2-10 title="Fila 10 · ¿El COPY sale de la caché?"}
```bash
docker build -t saludo:1 .
```
:::

::: hint {of="xd-2-10"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm saludo:1` | `nube` | `sol` | pa: capa `adios`, imagen `hola` | `sol` | **`sol`** |

¿Cambió tu carpeta desde el último build?
:::

::: answer {of="xd-2-10"}
**Se rehace.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `build` | `nube` | **`nube`** | pa: capa `adios`, imagen `hola` | `sol` | — |

El build no imprime ningún saludo: el `COPY saludo.txt .` **se rehace**. Tu carpeta dice `nube` y el último build copió `sol`. El `COPY` lee un archivo distinto, así que la caché ya no sirve.

```text
$ docker build --progress=plain -t saludo:1 .
…
#6 [2/3] WORKDIR /s
#6 CACHED

#7 [3/3] COPY saludo.txt .
#7 DONE 0.0s
```

**Ojo al reproducirlo:** Si ya corriste esta secuencia, los builds de las filas 4 y 10 pueden salir `CACHED`. La caché se busca por contenido, y un build viejo con `sol` o `nube` sigue guardado.
:::

::: problem {#xd-2-11 title="Fila 11 · ¿Qué imprime cada run? (dos casillas)"}
```bash
docker run --rm saludo:1 sh -c 'echo x > saludo.txt; cat saludo.txt'
docker run --rm saludo:1
```
:::

::: hint {of="xd-2-11"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `build` | `nube` | `saludo:1` = el build de la fila 10 | pa: capa `adios`, imagen `hola` | `sol` | — |

Cada `docker run` crea un contenedor con capa nueva. ¿Qué dice la imagen después de la fila 10?
:::

::: answer {of="xd-2-11"}
- **Primero: `x`.**
- **Segundo: `nube`.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm … 'echo x …; cat …'` | `nube` | `nube` | pa: capa `adios`, imagen `hola` | `sol` | **`x`** |
| `run --rm saludo:1` | `nube` | `nube` | pa: capa `adios`, imagen `hola` | `sol` | **`nube`** |

En el primero, lo que va después de la imagen reemplaza al `CMD`. Ese contenedor escribe `x` en su propia capa y lo lee. `--rm` lo borra con todo y capa. El segundo es un contenedor nuevo, con capa vacía: lee la imagen, que dice `nube` desde la fila 10.
:::

::: problem {#xd-2-12 title="Fila 12 · adios y luna, ¿dónde existen?"}
```bash
docker rm -f pa
```

Opciones: tu carpeta, la imagen, un volumen, en ningún lado.
:::

::: hint {of="xd-2-12"}
Estado antes de esta fila:

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `run --rm saludo:1` | `nube` | `nube` | pa: capa `adios`, imagen `hola` | `sol` | **`nube`** |

¿Dónde vivía cada palabra, y qué comando borró ese lugar?
:::

::: answer {of="xd-2-12"}
**En ningún lado, los dos.**

| Paso | Tu carpeta | Imagen `saludo:1` | Contenedores | Volumen `caja` | Imprime |
|---|---|---|---|---|---|
| `rm -f pa` | `nube` | `nube` | **·** | `sol` |  |

Ninguna celda dice `adios` ni `luna`:

- `adios` vivía en la capa de pa. `docker rm -f pa` borró el contenedor con su capa.
- `luna` vivía en el primer `caja`. `docker volume rm caja` lo borró en la fila 7. El `caja` de hoy es otro y dice `sol`.

```text
$ docker run --rm -v caja:/s saludo:1
sol
$ cat saludo.txt
nube
```
:::

**Para recordar:**

- `stop` conserva la capa del contenedor; `rm` la borra.
- Un volumen vacío se llena con la imagen; uno con contenido tapa a la imagen.
- Un build nuevo no cambia los contenedores que ya existen.

**Repasa:** [[ciclo-de-vida-de-un-contenedor]], [[named-volumes-y-postgres]], [[donde-vive-cada-byte]] y [[los-ocho-casos]].

## Parte 3 · Lee el error

Cada caso trae el comando y lo que contestó Docker. Escribe **qué pasó** y **qué harías**.

::: problem {#xd-e1 title="Error 1 · Correr dos veces lo mismo"}
Justo después de la fila 1, alguien vuelve a correr la misma línea:

```text
$ docker run -d --name pa saludo:1 sleep 600
docker: Error response from daemon: Conflict. The container name "/pa" is already in use by container "94ec18da7a0a…". You have to remove (or rename) that container to be able to reuse that name.
```
:::

::: hint {of="xd-e1"}
Lee la palabra clave del mensaje: *Conflict*. ¿Qué ya existe?
:::

::: answer {of="xd-e1"}
**Qué pasó.** Ya existe un contenedor llamado `pa`, y los nombres no se repiten. Docker no crea nada y sale con código 125.

| Paso | Contenedor `pa` | Su capa |
|---|---|---|
| después de la fila 1 | corriendo | `adios` |
| `docker run -d --name pa …` otra vez | corriendo, **el mismo**: el nuevo no se crea | `adios` |
| `docker rm -f pa` | **·** | **·** |
| `docker run -d --name pa …` | **nuevo**, corriendo | **vacía** |

**Qué harías.** Si no lo necesitas, `docker rm -f pa` y vuelves a correr; ojo, con él se va su capa y el `adios`. Si sí lo necesitas, usa otro nombre en `--name`.
:::

::: problem {#xd-e2 title="Error 2 · Un build que no arranca"}
```text
$ docker build -t saludo:1
ERROR: docker: 'docker buildx build' requires 1 argument

Usage:  docker buildx build [OPTIONS] PATH | URL | -
```
:::

::: hint {of="xd-e2"}
Compara con `docker build -t saludo:1 .`. ¿Qué falta?
:::

::: answer {of="xd-e2"}
**Qué pasó.** Falta el contexto, el `.` final: el build no sabe qué carpeta mandar.

**Qué harías.** `docker build -t saludo:1 .`
:::

::: problem {#xd-e3 title="Error 3 · Exec a un contenedor detenido"}
Después de `docker stop pa`:

```text
$ docker exec pa cat saludo.txt
Error response from daemon: container 94ec18da7a0a… is not running
```
:::

::: hint {of="xd-e3"}
¿En qué estado quedó pa después de `stop`? ¿En cuál tiene que estar para un `exec`?
:::

::: answer {of="xd-e3"}
**Qué pasó.** `exec` entra a un contenedor **corriendo**, y pa está detenido.

| Paso | Estado de `pa` | Su capa | `docker exec pa cat saludo.txt` |
|---|---|---|---|
| `docker stop pa` | **detenido** | `adios` | falla: `is not running` |
| `docker start pa` | **corriendo** | `adios` | imprime `adios` |

**Qué harías.** `docker start pa` y luego el `exec`. Su capa sigue ahí, porque `stop` no la borra.
:::

::: problem {#xd-e4 title="Error 4 · Una terminal que no abre"}
```text
$ docker run --rm -it saludo:1 bash
docker: Error response from daemon: failed to create task for container: … exec: "bash": executable file not found in $PATH
```
:::

::: hint {of="xd-e4"}
¿Qué imagen base usa `saludo:1`, y qué shell trae esa imagen?
:::

::: answer {of="xd-e4"}
**Qué pasó.** La imagen es Alpine, y Alpine no trae `bash`: trae `sh`.

**Qué harías.** `docker run --rm -it saludo:1 sh`.
:::

::: problem {#xd-e5 title="Error 5 · Copiar desde fuera del contexto"}
Un Dockerfile en `proyecto/app/` tiene la línea `COPY ../datos /datos`, y se construye desde `proyecto/app/`:

```text
$ docker build -t app:1 .
ERROR: failed to build: failed to solve: failed to compute cache key: … "/datos": not found
```
:::

::: hint {of="xd-e5"}
¿Desde qué carpeta lee el build? ¿Puede un `COPY` salir de ella?
:::

::: answer {of="xd-e5"}
**Qué pasó.** El contexto es `proyecto/app/`, y un `COPY` no puede salir de él: Docker busca `datos` **dentro** del contexto, y ahí no está.

**Qué harías.** Construir desde `proyecto/` con `docker build -t app:1 -f app/Dockerfile .`, y cambiar la línea a `COPY datos /datos`. O mejor, si los datos son grandes o cambian seguido, no copiarlos: montarlos al correr.
:::

**Repasa:** [[anatomia-de-docker-run]], [[ciclo-de-vida-de-un-contenedor]] y [[las-cuatro-trampas]].
