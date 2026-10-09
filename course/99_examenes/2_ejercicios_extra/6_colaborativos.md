---
id: ejercicios-colaborativos
title: "Ejercicios colaborativos: GitHub y Docker en equipo"
nav_title: "Colaborativos: GitHub y Docker"
summary: "Dos ejercicios largos donde tres personas trabajan el mismo repositorio a la vez: pushes rechazados, conflictos, merges limpios que rompen, y lo que eso le hace a cada imagen, contenedor y volumen. Con pista y respuesta plegadas."
status: ready
estimated_time: 90m
tags: [ejercicios, git, github, merge, conflictos, docker, volumenes, postgres, equipo]
prerequisites: [ejercicios-github, ejercicios-docker]
---

# Ejercicios colaborativos: GitHub y Docker en equipo

**[PDF sin respuestas, para imprimir](../_assets/practica-colaborativos.pdf)** · unos 90 minutos · sin apuntes

En los parciales, una sola persona cambiaba un archivo y seguías qué pasaba. Aquí trabajan **tres a la vez**: Ana, Beto y Caro. Cada quien tiene su clon, su máquina, sus imágenes, sus contenedores y sus volúmenes. Los cambios llegan de una máquina a otra **sólo** a través de GitHub, con `push` y `pull`.

Debajo de cada pregunta hay una **pista** y la **respuesta**, plegadas. Abre la pista sólo si llevas un rato atorado.

Los dos escenarios se reprodujeron completos: un repositorio local hizo de GitHub, hubo tres clones, y cada comando corrió con Git 2.34 y Docker 29.6. Las salidas que ves son las reales.

**Cómo leerlos.** Cada fila dice quién actúa y en qué orden; cada fila parte de lo que dejaron las anteriores. Las preguntas usan sólo comandos que ya viste en los parciales. Al final de cada ejercicio no se piden comandos: se pregunta **qué práctica** lo habría evitado.

**Una diferencia con el curso.** Aquí el equipo comparte **un solo repositorio** donde los tres pueden escribir, como en un proyecto de trabajo. En el curso cada quien tiene su fork y propone por pull request. Con un repositorio compartido aparece algo que con forks casi no pasa: dos personas empujando a la misma rama.

## Ejercicio 1 · El umbral

### El punto de partida

El equipo mantiene `alertas/`, un servicio que imprime el umbral con el que filtra. En GitHub, `main` tiene un solo commit, **B**:

```text
alertas/
├── Dockerfile
├── alerta.sh
└── config.env
```

`config.env`:

```text
UMBRAL=5
```

`alerta.sh` carga ese archivo e imprime el valor:

```bash
. ./config.env
echo "umbral: $UMBRAL"
```

`Dockerfile`: la configuración se **copia dentro de la imagen**.

```dockerfile
FROM alpine:3.20
WORKDIR /app
COPY config.env alerta.sh ./
CMD ["sh", "alerta.sh"]
```

Los tres ya clonaron el repositorio y, cada uno en su máquina, corrieron:

```bash
docker build -t alertas:1 .
docker run -d --name alerta alertas:1 sleep 600
```

Así que cada quien tiene una imagen `alertas:1` y un contenedor `alerta` encendido, los dos con `UMBRAL=5`. Ningún `sleep 600` termina durante el ejercicio.

### La secuencia

::: problem {#xco1-1 title="Fila 1 · Ana cambia y sube"}
```bash
# Ana
echo "UMBRAL=10" > config.env
git commit -am "umbral 10"
git push origin main
docker run --rm alertas:1
```

¿Qué `UMBRAL` hay ahora en GitHub, en el clon de Beto y en la imagen de Ana? ¿Qué imprime el `docker run` de Ana?
:::

::: hint {of="xco1-1"}
Estado antes de la fila 1:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | `B` | `B` | `B` | `B` |
| `config.env` | `5` | `5` | `5` | `5` |
| Imagen `alertas:1` | · | build de `B` | build de `B` | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

Un `push` actualiza GitHub. ¿Qué celda de la tabla cambia con un `build`, y lo corrió Ana?
:::

::: answer {of="xco1-1"}
**GitHub: 10. Clon de Beto: 5. Imagen de Ana: 5. El run imprime `umbral: 5`.**

Estado después de la fila 1:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«umbral 10»** | **«umbral 10»** | `B` | `B` |
| `config.env` | **`10`** | **`10`** | `5` | `5` |
| Imagen `alertas:1` | · | build de `B` | build de `B` | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

- El push copia el commit a GitHub y a nada más. Beto y Caro lo reciben sólo cuando hagan `pull`.
- La imagen de Ana sigue siendo el build de `B`. Commitear y subir no reconstruye: la imagen sólo cambia con `docker build`.

```text
$ docker run --rm alertas:1
umbral: 5
```
:::

::: problem {#xco1-2 title="Fila 2 · Ana reconstruye"}
```bash
# Ana
docker build -t alertas:1 .
docker run --rm alertas:1
docker exec alerta sh alerta.sh
```

¿Qué imprime el run? ¿Y el exec?
:::

::: hint {of="xco1-2"}
Estado antes de la fila 2:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«umbral 10»** | **«umbral 10»** | `B` | `B` |
| `config.env` | **`10`** | **`10`** | `5` | `5` |
| Imagen `alertas:1` | · | build de `B` | build de `B` | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

¿De qué build nace el contenedor del `run`? ¿Y de cuál nació `alerta`?
:::

::: answer {of="xco1-2"}
**run: `umbral: 10`. exec: `umbral: 5`.**

Estado después de la fila 2:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | `B` | `B` |
| `config.env` | `10` | `10` | `5` | `5` |
| Imagen `alertas:1` | · | **build de «umbral 10»** | build de `B` | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

- El run crea un contenedor nuevo de la imagen recién construida, que copió el `config.env` del disco (10).
- El exec entra a `alerta`, que nació de la imagen de `B`. Reconstruir una imagen no cambia los contenedores que ya existen.

Ana tiene ahora en su máquina dos versiones: su imagen (10) y su contenedor `alerta` (5).
:::

::: problem {#xco1-3 title="Fila 3 · Beto, sin haber hecho pull"}
```bash
# Beto
echo "UMBRAL=20" > config.env
git commit -am "umbral 20"
git push origin main
```

Git le contesta:

```text
 ! [rejected]        main -> main (fetch first)
hint: Updates were rejected because the remote contains work that you do
```

¿Por qué lo rechaza? ¿Qué `UMBRAL` hay en GitHub? Si ahora Beto corre `docker build -t alertas:1 .` y `docker run --rm alertas:1`, ¿qué imprime?
:::

::: hint {of="xco1-3"}
Estado antes de la fila 3:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | `B` | `B` |
| `config.env` | el de la fila 1 | `10` | `5` | `5` |
| Imagen `alertas:1` | · | **build de «umbral 10»** | build de `B` | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

¿Qué commit tiene GitHub que Beto no tiene? ¿De dónde copia el build: de GitHub o del disco de Beto?
:::

::: answer {of="xco1-3"}
**Rechazado. GitHub sigue en `UMBRAL=10`. El run de Beto imprime `umbral: 20`.**

Estado después de la fila 3:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | **«umbral 20» (sin subir)** | `B` |
| `config.env` | `10` | `10` | **`20`** | `5` |
| Imagen `alertas:1` | · | build de «umbral 10» | **build de «umbral 20»** | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

Las dos historias salieron de `B` y ya no son una la continuación de la otra. Con `D` = «umbral 10» (Ana) y `V` = «umbral 20» (Beto):

```text
B---D      main en GitHub
 \
  V        main de Beto
```

- GitHub tiene `D` y Beto no. Si Git aceptara el push, `main` en GitHub pasaría a `B---V` y `D` desaparecería. Git obliga a **integrar primero**.
- El push no entró, así que GitHub sigue en `D` (10).
- El build copia **el disco de Beto** (20), no GitHub. Beto tiene una imagen con un valor que no existe en ningún otro lado.
:::

::: problem {#xco1-4 title="Fila 4 · Caro se pone al día"}
```bash
# Caro
git pull origin main
docker run --rm alertas:1
docker run --rm -v "$(pwd)/config.env":/app/config.env alertas:1
```

¿Qué hace el pull? ¿Qué imprime cada run?
:::

::: hint {of="xco1-4"}
Estado antes de la fila 4:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | **«umbral 20» (sin subir)** | `B` |
| `config.env` | `10` | `10` | **`20`** | `5` |
| Imagen `alertas:1` | · | build de «umbral 10» | **build de «umbral 20»** | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

¿Tiene Caro commits que GitHub no tenga? ¿Qué celdas de Caro puede cambiar un `pull`, y cuáles no? ¿Qué tapa un montaje?
:::

::: answer {of="xco1-4"}
**El pull es un fast-forward. Primer run: `umbral: 5`. Segundo run: `umbral: 10`.**

Estado después de la fila 4:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | «umbral 20» (sin subir) | **«umbral 10»** |
| `config.env` | `10` | `10` | `20` | **`10`** |
| Imagen `alertas:1` | · | build de «umbral 10» | build de «umbral 20» | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

- Caro no tenía commits propios, así que su `main` sólo avanza hasta «umbral 10». Git lo dice: `Fast-forward`.
- El primer run usa su imagen, que sigue siendo el build de `B`: un pull no reconstruye.
- El segundo run monta su `config.env` del disco (10) encima del de la imagen.

Mismo commit en el disco, dos resultados distintos según **cómo** corra el contenedor.
:::

::: problem {#xco1-5 title="Fila 5 · Beto integra"}
```bash
# Beto
git pull origin main
```

Git contesta:

```text
CONFLICT (content): Merge conflict in config.env
Automatic merge failed; fix conflicts and then commit the result.
```

¿Cómo se ve ahora `config.env` en el disco de Beto? ¿Qué imprime `docker exec alerta sh alerta.sh`? ¿Y `docker run --rm alertas:1`?
:::

::: hint {of="xco1-5"}
Estado antes de la fila 5:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | «umbral 20» (sin subir) | **«umbral 10»** |
| `config.env` | `10` | `10` | `20` | **`10`** |
| Imagen `alertas:1` | · | build de «umbral 10» | build de la fila 3 | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

Los dos cambiaron la misma línea desde `B`. ¿Qué celdas toca un `pull`? ¿Cuáles se construyeron antes?
:::

::: answer {of="xco1-5"}
**`config.env` queda con marcadores. exec: `umbral: 5`. run: `umbral: 20`.**

Estado después de la fila 5:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | **«umbral 20», merge a medias** | «umbral 10» |
| `config.env` | `10` | `10` | **marcadores 20/10** | `10` |
| Imagen `alertas:1` | · | build de «umbral 10» | build de «umbral 20» | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

El archivo en el disco de Beto (salida real):

```text
<<<<<<< HEAD
UMBRAL=20
=======
UMBRAL=10
>>>>>>> 7624c311a90082e87f22ea92fd5ed4d1037efa0b
```

- Arriba de `=======` está `HEAD`: el commit de Beto (20).
- Abajo está lo que llegó de GitHub: el 10 de Ana.
- El hash después de `>>>>>>>` es el del commit «umbral 10» que el pull trajo. Con `git pull origin main` Git no pone un nombre de rama sino ese hash completo. `git log --oneline origin/main` lo muestra abreviado: `7624c31 umbral 10`. En tu máquina el hash será otro, porque depende del autor y de la fecha del commit.
- `git status --short` marca el archivo `UU config.env`: el merge no se ha terminado.
- Sale un merge con marcadores, y no otra cosa, porque el curso configuró `git config --global pull.rebase false` ([[clonar-y-actualizar]]).

Los contenedores no se enteran:

- El exec entra a `alerta`, que nació de `B`.
- El run usa la imagen que Beto construyó en la fila 3 (20).

El conflicto está **sólo en el disco**.
:::

::: problem {#xco1-6 title="Fila 6 · Beto «termina»"}
```bash
# Beto
git add config.env
git commit -m "listo"
docker build -t alertas:1 .
docker run --rm alertas:1
echo $?
git push origin main
```

Beto no editó el archivo. ¿Git acepta el commit? ¿El build termina bien? ¿Qué imprime el run y qué da `echo $?`? ¿GitHub acepta el push?
:::

::: hint {of="xco1-6"}
Estado antes de la fila 6:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «umbral 10» | «umbral 10» | **«umbral 20», merge a medias** | «umbral 10» |
| `config.env` | `10` | `10` | **marcadores 20/10** | `10` |
| Imagen `alertas:1` | · | build de «umbral 10» | build de «umbral 20» | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

¿Revisa `git add` el contenido del archivo? ¿Le importa a `COPY`? ¿Qué hace `sh` con una línea que empieza con `<<<`?
:::

::: answer {of="xco1-6"}
**Git acepta el commit. El build termina bien. El run truena y `echo $?` da 2. GitHub acepta el push.**

Estado después de la fila 6:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«listo»** | «umbral 10» | **«listo»** | «umbral 10» |
| `config.env` | **marcadores 20/10** | `10` | marcadores 20/10 | `10` |
| Imagen `alertas:1` | · | build de «umbral 10» | **build de «listo»** | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

- `git add` le dice a Git que el conflicto está resuelto. Git no revisa si quedaron marcadores.
- `COPY` copia bytes sin leerlos, así que el build no falla.
- `sh` carga `config.env` y lee `<<<<<<<` como una redirección: código 2.
- El push entra porque el `main` de Beto ya contiene `D`: subir no borra nada.

```text
$ docker run --rm alertas:1
alerta.sh: ./config.env: line 1: syntax error: unexpected redirection
$ echo $?
2
```

La historia, con `M` = «listo»:

```text
B---V---M      main de Beto y main en GitHub
 \     /
  D----
```

GitHub no revisa contenido: **el archivo roto ya está en `main`**. Beto tuvo tres avisos y no miró ninguno: el `CONFLICT`, el `UU` y su propio contenedor tronando.
:::

::: problem {#xco1-7 title="Fila 7 · Caro vuelve a ponerse al día"}
```bash
# Caro
git pull origin main
docker run --rm alertas:1
docker run --rm -v "$(pwd)/config.env":/app/config.env alertas:1
```

¿Qué hace el pull? ¿Qué imprime cada run?
:::

::: hint {of="xco1-7"}
Estado antes de la fila 7:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«listo»** | «umbral 10» | **«listo»** | «umbral 10» |
| `config.env` | **marcadores 20/10** | `10` | marcadores 20/10 | `10` |
| Imagen `alertas:1` | · | build de «umbral 10» | **build de «listo»** | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

¿Tiene Caro commits propios? ¿Qué trae ahora `main` en `config.env`? ¿Qué copia usa cada run?
:::

::: answer {of="xco1-7"}
**Fast-forward. Primer run: `umbral: 5`. Segundo run: truena con código 2.**

Estado después de la fila 7:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «listo» | «umbral 10» | «listo» | **«listo»** |
| `config.env` | marcadores 20/10 | `10` | marcadores 20/10 | **marcadores 20/10** |
| Imagen `alertas:1` | · | build de «umbral 10» | build de «listo» | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

- Caro sigue sin commits propios: su `main` avanza de «umbral 10» a «listo». Caro no hizo nada mal, y aun así su disco tiene el archivo roto.
- El primer run usa su imagen, el build de `B`. Por eso **parece que todo funciona**.
- El segundo run monta el archivo con marcadores y truena con el mismo error:

```text
alerta.sh: ./config.env: line 1: syntax error: unexpected redirection
```

Si Caro sólo probara sin montaje, tardaría en descubrirlo. Si reconstruye, su imagen también queda rota.
:::

::: problem {#xco1-8 title="Fila 8 · Ana lo arregla"}
El equipo acuerda un umbral de 15.

```bash
# Ana
git pull origin main
echo "UMBRAL=15" > config.env
git commit -am "umbral acordado 15"
git push origin main
docker build -t alertas:1 .
docker run --rm alertas:1
```

¿Cómo sale el pull? ¿Qué imprime el run?
:::

::: hint {of="xco1-8"}
Estado antes de la fila 8:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «listo» | «umbral 10» | «listo» | **«listo»** |
| `config.env` | marcadores 20/10 | `10` | marcadores 20/10 | **marcadores 20/10** |
| Imagen `alertas:1` | · | build de «umbral 10» | build de «listo» | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

¿Tiene Ana commits que GitHub no tenga?
:::

::: answer {of="xco1-8"}
**El pull es un fast-forward. El run imprime `umbral: 15`.**

Estado después de la fila 8:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«umbral acordado 15»** | **«umbral acordado 15»** | «listo» | «listo» |
| `config.env` | **`15`** | **`15`** | marcadores 20/10 | marcadores 20/10 |
| Imagen `alertas:1` | · | **build de «umbral acordado 15»** | build de «listo» | build de `B` |
| Contenedor `alerta` | · | nació de `B` | nació de `B` | nació de `B` |

- Ana no tenía commits nuevos: su `main` avanza a «listo» y su disco recibe los marcadores.
- `echo "UMBRAL=15" > config.env` reemplaza el archivo completo, marcadores incluidos.
- El push entra sin rechazo, porque Ana partió del último commit de GitHub. `main` en GitHub vuelve a estar sano.
- El build copia el disco (15).
:::

::: problem {#xco1-9 title="Fila 9 · La foto final"}
Nadie más hace nada. Llena la tabla con lo que dice cada lugar: un número, «marcadores» o «truena».

| Lugar | ¿Qué dice? |
|---|---|
| `main` en GitHub | |
| Disco de Ana | |
| Disco de Beto | |
| Disco de Caro | |
| Imagen de Ana | |
| Imagen de Beto | |
| Imagen de Caro | |
| Contenedor `alerta` de Ana | |
| Contenedor `alerta` de Beto | |
| Contenedor `alerta` de Caro | |
:::

::: hint {of="xco1-9"}
Estado antes de la fila 9 (nadie hace nada más, así que es también el final):

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«umbral acordado 15»** | **«umbral acordado 15»** | «listo» | «listo» |
| Imagen `alertas:1` | · | build de la fila 8 | build de la fila 6 | build del inicio |
| Contenedor `alerta` | · | del inicio | del inicio | del inicio |

Traduce cada celda: ¿qué valor tenía el disco en el commit de cada build? ¿De qué build nació cada `alerta`?
:::

::: answer {of="xco1-9"}
Estado: igual que en la fila 8.

| Lugar | ¿Qué dice? | Por qué |
|---|---|---|
| `main` en GitHub | 15 | El arreglo de Ana |
| Disco de Ana | 15 | Lo escribió ella |
| Disco de Beto | marcadores | No ha hecho pull desde su commit roto |
| Disco de Caro | marcadores | Su último pull fue el de la fila 7 |
| Imagen de Ana | 15 | Reconstruyó en la fila 8 |
| Imagen de Beto | truena | La construyó con marcadores en la fila 6 |
| Imagen de Caro | 5 | Nunca reconstruyó |
| Contenedor `alerta` de Ana | 5 | Nació al principio |
| Contenedor `alerta` de Beto | 5 | Nació al principio |
| Contenedor `alerta` de Caro | 5 | Nació al principio |

**Cuatro valores distintos para la misma variable**, y el que está en GitHub (15), fuera de GitHub, sólo lo tiene la imagen de Ana. Si Beto hace pull y reconstruye, su imagen da 15, pero su contenedor `alerta` sigue en 5 hasta que lo borre y lo cree otra vez.
:::

### Diagnóstico y prácticas

Aquí no se piden comandos: se pide entender qué falló y qué costumbre del equipo lo habría evitado.

::: problem {#xco1-d1 title="D1 · ¿Dónde entró el error?"}
¿En qué fila entró el archivo roto a `main`? ¿Quién pudo detenerlo antes, y con qué señal?
:::

::: hint {of="xco1-d1"}
¿En qué fila aceptó GitHub un commit con marcadores? ¿Cuántos avisos vio Beto antes?
:::

::: answer {of="xco1-d1"}
**En la fila 6**, con el push de Beto. Beto tuvo tres señales:

1. el mensaje `CONFLICT` de la fila 5;
2. el `UU` de `git status`;
3. su propio contenedor tronando con código 2, **antes** de hacer push.

Cualquiera de las tres bastaba para no subir. Nadie más pudo detenerlo: el equipo subía directo a `main`, sin una revisión en medio.
:::

::: problem {#xco1-d2 title="D2 · ¿Por qué cada quien ve otra cosa?"}
Al final hay cuatro valores distintos de `UMBRAL` en el equipo. Explica las **tres** razones por las que una persona puede estar corriendo una versión distinta de la que hay en GitHub.
:::

::: hint {of="xco1-d2"}
¿Cuántas copias hay entre GitHub y lo que imprime un contenedor?
:::

::: answer {of="xco1-d2"}
Entre GitHub y lo que imprime un contenedor hay **tres copias**, y cada una se actualiza por separado:

1. **El disco** sólo cambia con `pull`. Si no lo haces, trabajas sobre lo viejo, como Beto en la fila 3.
2. **La imagen** sólo cambia con `build`, y copia **el disco**, no GitHub. Con cambios sin subir, o con un archivo roto en el disco, tu imagen no es igual a la de nadie.
3. **El contenedor** se queda con la imagen con la que nació. Reconstruir no lo actualiza: hay que borrarlo y crear otro.

«En mi máquina funciona» suele significar que una de esas tres copias está vieja.
:::

::: problem {#xco1-d3 title="D3 · Las prácticas que faltaron"}
Sin escribir comandos: ¿qué cuatro o cinco costumbres de equipo habrían evitado este ejercicio completo? Para cada una, di qué fila habría cambiado.
:::

::: hint {of="xco1-d3"}
Repasa las filas 3, 5, 6, 7 y 9: ¿qué debió pasar antes de cada una?
:::

::: answer {of="xco1-d3"}
Cualquiera de éstas, bien ligada a una fila, vale:

| Práctica | Qué habría cambiado |
|---|---|
| **Ponerse al día antes de trabajar** | Beto habría partido del 10 de Ana: sin rechazo ni conflicto (filas 3 y 5) |
| **Ramas y pull requests** | El commit roto se queda en una rama y alguien lo ve antes de que llegue a Caro (filas 6 y 7) |
| **Revisar el conflicto** | Abrir el archivo, buscar marcadores y preguntar qué valor gana: Beto no commitea marcadores (fila 6) |
| **Probar antes de subir** | Una revisión automática que construya y **arranque** la imagen en cada pull request: el código 2 bloquea el merge (fila 6) |
| **Configuración fuera de la imagen** | Montada o por variable de entorno: cambiar un número no exige reconstruir (filas 2, 4 y 9) |
| **Imagen con el commit en el nombre** | En lugar de `alertas:1` siempre, y recreando contenedores: se sabe qué versión corre cada uno (fila 9) |
| **Acordar quién decide un valor** | El 10 contra el 20 era un desacuerdo de personas, no de Git |
:::

## Ejercicio 2 · La base de datos

### El punto de partida

El equipo mantiene `tienda/`, con una base Postgres. Los cambios a la base son archivos SQL versionados en Git:

```text
tienda/
└── sql/
    └── 001_clientes.sql
```

`001_clientes.sql`:

```sql
CREATE TABLE clientes (id serial PRIMARY KEY, nombre text);
```

Cada quien levanta la base en su máquina, desde la carpeta del repo, siempre con el mismo comando:

```bash
docker run -d --name db -e POSTGRES_PASSWORD=fdd \
  -v datos:/var/lib/postgresql/data \
  -v "$(pwd)/sql":/docker-entrypoint-initdb.d \
  postgres:16
```

- `datos` es un **named volume**: ahí vive la base, y sobrevive a `docker rm`.
- El segundo `-v` monta la carpeta `sql/` del repo donde la imagen busca scripts de arranque.

**Un dato que necesitas.** Al arrancar, la imagen de Postgres:

- corre los `.sql` de `/docker-entrypoint-initdb.d`, en orden alfabético, **sólo si el volumen de datos está vacío**;
- si el volumen ya tiene una base, se salta todos los scripts y arranca;
- si un script falla, se detiene, y el contenedor se apaga.

Para ver las tablas: `docker exec db psql -U postgres -c '\dt'`.

Los tres clonaron el repo y levantaron su base. Cada quien tiene un volumen `datos` con la tabla `clientes`.

### La secuencia

::: problem {#xco2-1 title="Fila 1 · Ana agrega la tabla ventas, en una rama"}
```bash
# Ana
git switch -c ventas
# crea sql/002_ventas.sql:
#   CREATE TABLE ventas (id serial PRIMARY KEY,
#                        cliente int REFERENCES clientes(id), total numeric);
git add sql
git commit -m "tabla ventas"
git push -u origin ventas
docker rm -f db
docker run -d --name db ... postgres:16      # el mismo comando de arriba
docker exec db psql -U postgres -c '\dt'
```

¿Qué tablas ve Ana?
:::

::: hint {of="xco2-1"}
Estado antes de la fila 1. «inicial» es el commit con `001`. En la fila «Ramas», GitHub lista sus ramas; cada persona, la rama en la que está:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | `main` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | `001` | `001` | `001` |
| Contenedor `db` | · | encendido, inicio | encendido, inicio | encendido, inicio |
| Volumen `datos` | · | inicio: `001` | inicio: `001` | inicio: `001` |

¿`docker rm` borra el volumen? ¿Cuándo corre Postgres los scripts?
:::

::: answer {of="xco2-1"}
**Sólo `clientes`.**

Estado después de la fila 1:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | **`main`, `ventas`** | **en `ventas`** | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | **`001`, `002`** | `001` | `001` |
| Contenedor `db` | · | **encendido, fila 1** | encendido, inicio | encendido, inicio |
| Volumen `datos` | · | inicio: `001` | inicio: `001` | inicio: `001` |

- `docker rm` borra el contenedor, no el volumen.
- El volumen `datos` ya tenía una base, así que Postgres se salta **todos** los scripts, incluido el nuevo `002_ventas.sql`, aunque esté montado ahí mismo.

```text
$ docker logs db
PostgreSQL Database directory appears to contain a database; Skipping initialization
```
:::

::: problem {#xco2-2 title="Fila 2 · Ana prueba con un volumen nuevo"}
```bash
# Ana
docker rm -f db
docker volume rm datos
docker run -d --name db ... postgres:16
docker exec db psql -U postgres -c '\dt'
```

¿Qué tablas ve ahora? ¿Qué perdió?
:::

::: hint {of="xco2-2"}
Estado antes de la fila 2:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | **`main`, `ventas`** | **en `ventas`** | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | **`001`, `002`** | `001` | `001` |
| Contenedor `db` | · | **encendido, fila 1** | encendido, inicio | encendido, inicio |
| Volumen `datos` | · | inicio: `001` | inicio: `001` | inicio: `001` |

Con el volumen borrado, ¿cómo llega el siguiente `run` a `datos`?
:::

::: answer {of="xco2-2"}
**`clientes` y `ventas`. Perdió todo lo que tenía su base anterior.**

Estado después de la fila 2:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | `main`, `ventas` | en `ventas` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | `001`, `002` | `001` | `001` |
| Contenedor `db` | · | **encendido, fila 2** | encendido, inicio | encendido, inicio |
| Volumen `datos` | · | **fila 2: `001`, `002`** | inicio: `001` | inicio: `001` |

- `docker volume rm` borra el volumen y no tiene papelera.
- El `run` crea un volumen nuevo y vacío, así que corren `001` y `002`, en ese orden.

```text
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | clientes | table | postgres
 public | ventas   | table | postgres
(2 rows)
```

Satisfecha, Ana abre su pull request **#1** desde `ventas`.
:::

::: problem {#xco2-3 title="Fila 3 · Beto agrega reportes, en su rama"}
Al mismo tiempo, Beto necesita una tabla con ventas por mes:

```bash
# Beto
git switch -c reportes
# crea sql/003_ventas_mensuales.sql:
#   CREATE TABLE ventas (mes text PRIMARY KEY, total numeric);
git add sql
git commit -m "ventas mensuales"
git push -u origin reportes
docker rm -f db
docker volume rm datos
docker run -d --name db ... postgres:16
docker exec db psql -U postgres -c '\dt'
```

¿Qué tablas ve Beto?
:::

::: hint {of="xco2-3"}
Estado antes de la fila 3:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | `main`, `ventas` | en `ventas` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | `001`, `002` | `001` | `001` |
| Contenedor `db` | · | **encendido, fila 2** | encendido, inicio | encendido, inicio |
| Volumen `datos` | · | **fila 2: `001`, `002`** | inicio: `001` | inicio: `001` |

¿De qué commit nace `reportes`? ¿Está `002_ventas.sql` en la carpeta que monta Beto?
:::

::: answer {of="xco2-3"}
**`clientes` y `ventas`, pero su `ventas` tiene las columnas `mes` y `total`.**

Estado después de la fila 3:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | **`main`, `ventas`, `reportes`** | en `ventas` | **en `reportes`** | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | `001`, `002` | **`001`, `003`** | `001` |
| Contenedor `db` | · | encendido, fila 2 | **encendido, fila 3** | encendido, inicio |
| Volumen `datos` | · | fila 2: `001`, `002` | **fila 3: `001`, `003`** | inicio: `001` |

- `reportes` nació de «inicial», antes del trabajo de Ana. En el disco de Beto no existe `002_ventas.sql`.
- Su volumen es nuevo: corren `001` y `003`, y el `003` crea **su** `ventas`.

```text
$ docker exec db psql -U postgres -c '\d ventas'
               Table "public.ventas"
 Column |  Type   | Collation | Nullable | Default 
--------+---------+-----------+----------+---------
 mes    | text    |           | not null | 
 total  | numeric |           |          | 
```

En su máquina todo funciona, y abre su pull request **#2** desde `reportes`.
:::

::: problem {#xco2-4 title="Fila 4 · Se mergean los dos pull requests"}
En GitHub alguien mergea el #1 y luego el #2. ¿Hay conflicto? ¿Por qué? ¿Qué archivos hay ahora en `sql/` en `main`?
:::

::: hint {of="xco2-4"}
Estado antes de la fila 4:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «inicial» | «inicial» | «inicial» | «inicial» |
| Ramas | **`main`, `ventas`, `reportes`** | en `ventas` | **en `reportes`** | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001` | `001`, `002` | **`001`, `003`** | `001` |
| Contenedor `db` | · | encendido, fila 2 | **encendido, fila 3** | encendido, inicio |
| Volumen `datos` | · | fila 2: `001`, `002` | **fila 3: `001`, `003`** | inicio: `001` |

¿Tocaron Ana y Beto algún archivo en común?
:::

::: answer {of="xco2-4"}
**Sin conflicto. En `main` quedan `001_clientes.sql`, `002_ventas.sql` y `003_ventas_mensuales.sql`.**

Estado después de la fila 4:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«merge #2»** | «inicial» | «inicial» | «inicial» |
| Ramas | `main`, `ventas`, `reportes` | en `ventas` | en `reportes` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | **`001`, `002`, `003`** | `001`, `002` | `001`, `003` | `001` |
| Contenedor `db` | · | encendido, fila 2 | encendido, fila 3 | encendido, inicio |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | inicio: `001` |

Con `I` = «inicial», `V` = «tabla ventas», `R` = «ventas mensuales» y `M1`, `M2` = los dos merges:

```text
    V              ventas
   / \
  I---M1---M2      main
   \       /
    R------        reportes
```

- Ana creó `002_ventas.sql` y Beto creó `003_ventas_mensuales.sql`: **archivos distintos**. No hay ninguna línea que Git tenga que elegir.
- Comprobado: los dos merges salen con `Merge made by the 'ort' strategy.`, sin `CONFLICT`.
- Sólo cambió GitHub. Los discos, contenedores y volúmenes de los tres siguen igual.

Git no sabe SQL. No puede ver que **los dos archivos crean una tabla con el mismo nombre**.
:::

::: problem {#xco2-5 title="Fila 5 · Ana y Beto se ponen al día"}
Cada uno, en su máquina, con su volumen de la fila 2 y de la fila 3:

```bash
# Ana, y luego Beto
git switch main
git pull origin main
docker rm -f db
docker run -d --name db ... postgres:16
docker exec db psql -U postgres -c '\d ventas'
```

`\d ventas` muestra las columnas de la tabla. ¿Qué columnas ve cada uno? ¿Corrió algún script?
:::

::: hint {of="xco2-5"}
Estado antes de la fila 5:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«merge #2»** | «inicial» | «inicial» | «inicial» |
| Ramas | `main`, `ventas`, `reportes` | en `ventas` | en `reportes` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | **`001`, `002`, `003`** | `001`, `002` | `001`, `003` | `001` |
| Contenedor `db` | · | encendido, fila 2 | encendido, fila 3 | encendido, inicio |
| Volumen `datos` | · | de la fila 2 | de la fila 3 | inicio: `001` |

¿Están vacíos los volúmenes de Ana y de Beto?
:::

::: answer {of="xco2-5"}
**Ana ve `id`, `cliente`, `total`. Beto ve `mes`, `total`. No corrió ningún script.**

Estado después de la fila 5:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | **«merge #2»** | **«merge #2»** | «inicial» |
| Ramas | `main`, `ventas`, `reportes` | **en `main`** | **en `main`** | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | **`001`, `002`, `003`** | **`001`, `002`, `003`** | `001` |
| Contenedor `db` | · | **encendido, fila 5** | **encendido, fila 5** | encendido, inicio |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | inicio: `001` |

- El pull de cada uno es un fast-forward: ahora los dos montan `001`, `002` y `003`.
- Los dos volúmenes ya tenían base. Los dos logs dicen *Skipping initialization*.
- Cada `ventas` es la que dejó la inicialización de su volumen: la de Ana en la fila 2, la de Beto en la fila 3.

Los dos tienen el **mismo commit** en el disco y una tabla `ventas` **distinta** en la base. Y los dos dirían «en mi máquina funciona».
:::

::: problem {#xco2-6 title="Fila 6 · Caro se pone al día"}
Caro todavía tiene su volumen del principio.

```bash
# Caro
git pull origin main
docker rm -f db
docker run -d --name db ... postgres:16
docker exec db psql -U postgres -c '\dt'
```

¿Qué tablas ve?
:::

::: hint {of="xco2-6"}
Estado antes de la fila 6:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | **«merge #2»** | **«merge #2»** | «inicial» |
| Ramas | `main`, `ventas`, `reportes` | **en `main`** | **en `main`** | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | **`001`, `002`, `003`** | **`001`, `002`, `003`** | `001` |
| Contenedor `db` | · | **encendido, fila 5** | **encendido, fila 5** | encendido, inicio |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | del inicio |

¿Qué scripts corrieron en el volumen de Caro, y cuándo?
:::

::: answer {of="xco2-6"}
**Sólo `clientes`.**

Estado después de la fila 6:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | «merge #2» | «merge #2» | **«merge #2»** |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | **`001`, `002`, `003`** |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | **encendido, fila 6** |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | inicio: `001` |

Su volumen ya tenía base, así que no corre nada nuevo, aunque ahora monte los tres scripts. Caro tiene el código más reciente y la base más vieja del equipo.
:::

::: problem {#xco2-7 title="Fila 7 · Caro empieza desde cero"}
```bash
# Caro
docker rm -f db
docker volume rm datos
docker run -d --name db ... postgres:16
docker ps
docker logs db
```

¿Aparece `db` en `docker ps`? ¿Qué dice el log?
:::

::: hint {of="xco2-7"}
Estado antes de la fila 7:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | «merge #2» | «merge #2» | **«merge #2»** |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | **`001`, `002`, `003`** |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | **encendido, fila 6** |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | inicio: `001` |

Con el volumen vacío corren los tres scripts en orden. Cuando llega al `003`, ¿qué tabla ya existe?
:::

::: answer {of="xco2-7"}
**Una vez que termina la inicialización (unos segundos), `db` no aparece en `docker ps`: el contenedor se apagó con código 3. El log muestra que el `003` falló.**

Estado después de la fila 7:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | «merge #2» | «merge #2» | «merge #2» |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | **apagado, código 3** |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | **fila 7: `001`, `002`; `003` falló** |

`docker logs db` (salida real, sin las líneas de `initdb`):

```text
/usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/001_clientes.sql
CREATE TABLE
/usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/002_ventas.sql
CREATE TABLE
/usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/003_ventas_mensuales.sql
2026-10-09 18:43:29.201 UTC [63] ERROR:  relation "ventas" already exists
psql:/docker-entrypoint-initdb.d/003_ventas_mensuales.sql:1: ERROR:  relation "ventas" already exists
```

| Script | Resultado |
|---|---|
| `001` | Crea `clientes` |
| `002` | Crea `ventas` de Ana |
| `003` | Intenta crear `ventas` otra vez: error, y Postgres detiene la inicialización |

`docker ps -a` lo lista como `Exited (3)`. Es la primera vez que alguien prueba **el resultado del merge** sobre una base nueva, y truena. Cada rama por separado funcionaba.
:::

::: problem {#xco2-8 title="Fila 8 · Caro lo intenta otra vez"}
```bash
# Caro
docker rm -f db
docker run -d --name db ... postgres:16
docker exec db psql -U postgres -c '\dt'
```

No borró el volumen. ¿Arranca? ¿Qué tablas ve? ¿Por qué esto es **peor** que el error de la fila 7?
:::

::: hint {of="xco2-8"}
Estado antes de la fila 8:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | «merge #2» | «merge #2» | «merge #2» |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | **apagado, código 3** |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | de la fila 7 (arranque fallido) |

Después del intento fallido, ¿quedó vacío el volumen?
:::

::: answer {of="xco2-8"}
**Sí arranca, y ve `clientes` y `ventas`, la de Ana.**

Estado después de la fila 8:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | «merge #2» | «merge #2» | «merge #2» |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | **encendido, fila 8** |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | fila 7: `001`, `002`; `003` falló |

El volumen no quedó vacío: `001` y `002` se aplicaron antes de que fallara el `003`. Esta vez Postgres ve una base, se salta los scripts y arranca sin avisar. Salidas reales, recortadas:

```text
$ docker logs db
PostgreSQL Database directory appears to contain a database; Skipping initialization
...
$ docker exec db psql -U postgres -c '\dt'
 public | clientes | table | postgres
 public | ventas   | table | postgres
$ docker exec db psql -U postgres -c '\d ventas'
 id      | integer |           | not null | nextval('ventas_id_seq'::regclass)
 cliente | integer |           |          | 
 total   | numeric |           |          | 
```

**Es peor** porque ya no hay error que ver. Caro tiene una base sin la tabla de Beto, y nada se lo dice. El error de la fila 7 sólo quedaba en el log, y se fue con el `docker rm -f`.
:::

::: problem {#xco2-9 title="Fila 9 · La contraseña"}
Beto sube por error un archivo con la contraseña real de producción, y en el siguiente commit lo borra:

```bash
# Beto
echo "POSTGRES_PASSWORD=Tienda2026!" > .env
git add .env
git commit -m "config local"
git push origin main
rm .env
git commit -am "quita .env"
git push origin main
```

Alguien clona el repo después. ¿Ve `.env` en su disco? ¿La contraseña sigue en GitHub?
:::

::: hint {of="xco2-9"}
Estado antes de la fila 9:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | «merge #2» | «merge #2» | «merge #2» | «merge #2» |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | **encendido, fila 8** |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | fila 7: `001`, `002`; `003` falló |

¿Qué guarda Git de cada commit? ¿Un commit nuevo borra los anteriores?
:::

::: answer {of="xco2-9"}
**En el disco de un clon nuevo no hay `.env`. La contraseña sigue en GitHub, en el historial.**

Estado después de la fila 9:

| | GitHub | Ana | Beto | Caro |
|---|---|---|---|---|
| Último commit de `main` | **«quita .env»** | «merge #2» | **«quita .env»** | «merge #2» |
| Ramas | `main`, `ventas`, `reportes` | en `main` | en `main` | en `main` |
| `sql/` (lo que monta `$(pwd)/sql`) | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` | `001`, `002`, `003` |
| Contenedor `db` | · | encendido, fila 5 | encendido, fila 5 | encendido, fila 8 |
| Volumen `datos` | · | fila 2: `001`, `002` | fila 3: `001`, `003` | fila 7: `001`, `002`; `003` falló |

- El último commit, «quita .env», ya no tiene el archivo. Un clon nuevo no lo pone en el disco.
- El commit «config local» guarda el archivo completo, y sigue en `main`. Desde un clon nuevo:

```text
$ git show HEAD~1:.env
POSTGRES_PASSWORD=Tienda2026!
```

Borrar el archivo **no** borra el secreto. Lo único seguro es dar la contraseña por filtrada y **cambiarla**.
:::

### Diagnóstico y prácticas

::: problem {#xco2-d1 title="D1 · ¿Qué tiene cada base?"}
Después de la fila 8, ¿qué tablas y qué columnas de `ventas` tiene la base de Ana, la de Beto y la de Caro? ¿Cuál de las tres es «la correcta»?
:::

::: hint {of="xco2-d1"}
Junta las respuestas de las filas 5 y 8. ¿Qué scripts se aplicaron en cada volumen?
:::

::: answer {of="xco2-d1"}
| | Scripts aplicados | Tablas | Columnas de `ventas` |
|---|---|---|---|
| Ana | `001`, `002` (fila 2) | `clientes`, `ventas` | `id`, `cliente`, `total` |
| Beto | `001`, `003` (fila 3) | `clientes`, `ventas` | `mes`, `total` |
| Caro | `001`, `002`; el `003` corrió y falló (fila 7) | `clientes`, `ventas` | `id`, `cliente`, `total` |

**Ninguna es la correcta**, porque `main` no describe una base posible: sus scripts no pueden correr completos.

- La base de Caro tiene la misma estructura que la de Ana: el `003` sí corrió, pero falló sin dejar nada (log de la fila 7, `\d ventas` de la fila 8). Nada en la base registra que su inicialización falló.

Tres volúmenes, el mismo commit, dos estructuras de `ventas`. Cada volumen guarda **la historia de cuándo se creó**, no el código actual.
:::

::: problem {#xco2-d2 title="D2 · Sin conflicto, pero roto"}
Git dijo «sin conflicto» en la fila 4. Explica por qué eso no garantizaba nada.
:::

::: hint {of="xco2-d2"}
¿Qué compara Git cuando mezcla?
:::

::: answer {of="xco2-d2"}
Git compara **líneas de archivos**, no lo que significan. Un conflicto de Git aparece cuando dos personas cambian **las mismas líneas**. Aquí cambiaron archivos distintos, así que para Git no había nada que decidir.

El choque era de **significado**: dos archivos distintos crean la misma tabla. Es un conflicto que sólo aparece al **ejecutar** el resultado. «Se mezcla limpio» no es lo mismo que «funciona junto».
:::

::: problem {#xco2-d3 title="D3 · Las prácticas que faltaron"}
Sin escribir comandos: ¿qué costumbres habrían evitado los problemas de las filas 1, 5, 7, 8 y 9?
:::

::: hint {of="xco2-d3"}
Tres frentes: cómo se cambia una base que ya existe, qué se prueba antes de mergear, y qué nunca entra a Git.
:::

::: answer {of="xco2-d3"}
| Práctica | Qué habría cambiado |
|---|---|
| **Migraciones, no scripts de arranque** | Pasos numerados que se aplican una vez, en orden, y la base anota cuáles lleva: nadie borra su volumen (filas 1, 5 y 6) |
| **Probar el resultado del merge** | Una revisión automática levanta la base desde un volumen vacío con el `main` que resultaría: el choque de la fila 7 aparece en el pull request #2 |
| **Revisar lo que hace un pull request** | No basta con que mezcle limpio. Quien revisa el #2 pregunta si `ventas` ya existe |
| **Coordinar nombres y numeración** | Un solo lugar donde se ven los cambios a la base en curso: nadie inventa la misma tabla a la vez |
| **Leer el error antes de reintentar** | No reintentar sobre un volumen que quedó a medias: Caro no tapa el error (fila 8) |
| **Secretos fuera de Git** | `.env` en el `.gitignore` y un ejemplo sin valores reales en el repo; si uno se filtra, se cambia (fila 9) |
| **El volumen es estado** | Decidir en equipo cuándo se borra, y respaldar antes: Ana no pierde sus datos (fila 2) |
:::

**Repasa:** [[branches-y-merge]], [[el-flujo-del-curso]], [[named-volumes-y-postgres]], [[donde-vive-cada-byte]] y [[lo-que-no-se-sube]].
