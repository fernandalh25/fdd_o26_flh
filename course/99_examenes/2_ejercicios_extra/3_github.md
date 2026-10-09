---
id: ejercicios-github
title: "Ejercicios extra de GitHub"
nav_title: "GitHub"
summary: "Ejercicios nuevos con la forma del parcial de GitHub, cada uno con su respuesta explicada debajo: tres árboles de commits, siete errores reales de Git y un ritual con errores."
status: ready
estimated_time: 40m
tags: [ejercicios, git, github, ritual, branches, errores]
---

# Ejercicios extra de GitHub

**[PDF sin respuestas, para imprimir](../_assets/practica-github.pdf)** · unos 40 minutos · sin apuntes

Miden lo mismo que el parcial, con escenarios nuevos. Debajo de cada pregunta hay una **pista** y la **respuesta**, plegadas. Abre la pista sólo si llevas un rato atorado. Donde dice «con tus palabras», no escribas comandos. En `<login>` imagina el tuyo.

Casi todo sale de una idea: **un pull request muestra la diferencia entre tu rama y el punto donde se separó del `main` del curso**.

## Parte 1 · Lee el árbol

Los árboles se leen de izquierda a derecha: los commits viejos a la izquierda. Las letras son commits y los nombres de la derecha son ramas.

### Árbol 1 · Trabajó en main

Ana no creó rama: hizo sus commits M1 y M2 directamente en `main`, los subió a su fork y abrió el pull request desde `main`.

```text
o-o-o              upstream/main
     \
      M1---M2      main, origin/main
                   (pull request abierto desde main)
```

::: problem {#xg-1a title="Árbol 1a · ¿Qué muestra, y por qué sale en rojo?"}
¿Qué archivos muestra su pull request? Si todos están dentro de `estudiantes/ana/`, ¿por qué la revisión automática lo marca en rojo?
:::

::: hint {of="xg-1a"}
Además de los archivos, la revisión mira de qué rama sale el pull request.
:::

::: answer {of="xg-1a"}
**Muestra M1 y M2, que sólo tocan `estudiantes/ana/`. Sale en rojo porque el pull request sale desde `main`.**

La revisión no mira sólo los archivos: también mira la rama. Una de sus reglas rechaza el pull request que sale de la rama por defecto del fork. Tu `main` tiene que ser una copia limpia del curso, porque de ahí nacen todas tus tareas.
:::

::: problem {#xg-1b title="Árbol 1b · ¿Cómo queda su main después de actualizar?"}
La semana siguiente el curso publica un commit P, y Ana hace el bloque A completo del ritual. ¿Cómo queda su `main`? ¿Sigue siendo una copia del curso?
:::

::: hint {of="xg-1b"}
Después de que el curso publica P, ¿están `main` y `upstream/main` en la misma línea, o ya se separaron?
:::

::: answer {of="xg-1b"}
**Queda con un commit de merge (G) que junta todo lo del curso con M1 y M2. Ya no es una copia del curso, y no lo será mientras M1 y M2 no lleguen al curso.**

Antes del bloque A, con P ya publicado:

```text
o---o---o---P            upstream/main
         \
          M1---M2        main, origin/main
```

El bloque A, línea por línea:

| Línea | HEAD | `main` | `origin/main` | `upstream/main` |
|---|---|---|---|---|
| Inicio | `main` | M2 | M2 | último `o` |
| `git switch main` | `main` | M2 | M2 | último `o` |
| `git fetch upstream` | `main` | M2 | M2 | **P** |
| `git merge upstream/main` | `main` | **G** | M2 | P |
| `git push origin main` | `main` | G | **G** | P |

Después:

```text
o---o---o-----------P        upstream/main
         \           \
          M1---M2-----G      main, origin/main
```

**Por qué hay commit de merge.** Desde el último `o`, cada lado tiene commits que el otro no: `main` tiene M1 y M2, `upstream/main` tiene P. El merge no puede sólo avanzar a `main` hasta P, así que crea G, un commit con dos padres. Git lo dice: `Merge made by the 'ort' strategy.`

**El comando que lo prueba.** `git diff --stat upstream/main main` lista los archivos de M1 y M2. En una copia limpia del curso no listaría nada.
:::

::: problem {#xg-1c title="Árbol 1c · ¿Qué arrastra la tarea siguiente?"}
Desde ese `main` crea la rama de la tarea siguiente. Si el primer pull request todavía no se ha mergeado, ¿qué arrastra el nuevo?
:::

::: hint {of="xg-1c"}
Una rama hereda todo lo del commit donde nace. ¿Qué tiene ese `main`?
:::

::: answer {of="xg-1c"}
**Arrastra M1 y M2, y también el commit de merge G.**

La rama nace en G, y G contiene M1 y M2. El curso no los tiene. Por eso el pull request nuevo muestra los archivos de M1 y M2 además de los suyos. Es el mismo problema del examen A del parcial, sólo que aquí la herencia viene de `main`.

Comprobado: `git log --oneline upstream/main..<rama-nueva>` lista G, M2 y M1 antes de que la rama tenga un solo commit propio.
:::

::: problem {#xg-1d title="Árbol 1d · ¿Cómo lo arreglas sin perder M1 y M2?"}
Con tus palabras.
:::

::: hint {of="xg-1d"}
Primero pon el trabajo a salvo en un lugar que no sea `main`, y después limpia `main`.
:::

::: answer {of="xg-1d"}
**Primero pon M1 y M2 a salvo en una rama de tarea y súbela. Después regresa `main` a `upstream/main` y reemplaza con él el `main` del fork.**

Parto del estado de b): `main` y `origin/main` en G, `upstream/main` en P. `tarea-NN-nombre` es el nombre de la tarea a la que pertenecen M1 y M2.

| Paso | HEAD | `main` | `tarea-NN-nombre` | `origin/main` | `upstream/main` |
|---|---|---|---|---|---|
| Inicio | `main` | G | · | G | P |
| 1. `git switch -c tarea-NN-nombre` | **`tarea-NN-nombre`** | G | **G** | G | P |
| 2. `git push -u origin tarea-NN-nombre` | `tarea-NN-nombre` | G | G | G | P |
| 3. `git switch main` | **`main`** | G | G | G | P |
| 4. `git reset --hard upstream/main` | `main` | **P** | G | G | P |
| 5. `git push --force-with-lease origin main` | `main` | P | G | **P** | P |

Después del paso 2, `tarea-NN-nombre` ya está también en el fork. Entre el 2 y el 3, en GitHub, abre el pull request desde `tarea-NN-nombre` y cierra el que salía de `main`; eso no mueve ninguna ref.

Al final:

```text
o---o---o-----------P        main, origin/main, upstream/main
         \           \
          M1---M2-----G      tarea-NN-nombre
```

**Por qué el orden.** El paso 4 saca M1 y M2 de `main`, y `--hard` además reescribe tu disco. Sólo es seguro cuando ya existe otra rama que apunta a ellos (paso 1) y está en el fork (paso 2).

**Por qué hay que forzar el paso 5.** Un `git push origin main` normal lo rechaza Git:

```text
 ! [rejected]        main -> main (non-fast-forward)
```

El [[cheatsheet-git|cheatsheet]] dice «nunca `--force`» para este rechazo, y ésta es la única excepción: aquí quieres justo quitar G, M2 y M1 del `main` del fork, y no se pierden porque viven en `tarea-NN-nombre`. `--force-with-lease` sobrescribe el `main` del fork sólo si sigue en G, lo que tú viste por última vez; si alguien lo movió, se niega con `! [rejected] main -> main (stale info)`.

**Por qué aquí no sirve `git pull`:** traería de vuelta G, M2 y M1 a tu `main` y desharía el paso 4.
:::

### Árbol 2 · Dos ramas hermanas

Beto actualizó `main` y desde ahí creó las dos ramas, una para cada tarea. El pull request de la 07 sigue abierto.

```text
       A---B       tarea-07-git (PR abierto)
      /
o-o-o              main = upstream/main
      \
       D---E       tarea-08-datacamp-intro
```

::: problem {#xg-2a title="Árbol 2a · ¿Qué muestra el pull request de la 08?"}
¿Qué archivos muestra el pull request de `tarea-08-datacamp-intro`?
:::

::: hint {of="xg-2a"}
¿Dónde se separó la 08 de `main`? Cuenta lo que hay entre ese punto y E.
:::

::: answer {of="xg-2a"}
**Los archivos que cambian D y E, y nada más.**

La 08 se separó de `main` en el último `o`. Un pull request muestra lo que hay entre ese punto y la punta de la rama, y ahí sólo están D y E. A y B están en otra rama: no aparecen.

Comprobado: `git diff --stat main...tarea-08-datacamp-intro` lista sólo los archivos de D y E. Los tres puntos comparan contra el punto donde se separaron las ramas, igual que el pull request.
:::

::: problem {#xg-2b title="Árbol 2b · ¿Perdió la tarea 07?"}
Parado en `tarea-08-datacamp-intro`, Beto lista `estudiantes/beto/` y **no ve** `07_git/`. ¿Perdió su tarea 07? ¿Qué hizo `git switch` con esos archivos?
:::

::: hint {of="xg-2b"}
`git switch` cambia los archivos de tu disco por los de la rama a la que llegas.
:::

::: answer {of="xg-2b"}
**No perdió nada.** `git switch` pone en tu disco los archivos **de la rama a la que llegas**.

| Comando | Rama | `ls estudiantes/beto/` |
|---|---|---|
| `git switch tarea-08-datacamp-intro` | 08 | la carpeta de D y E; **no** `07_git/` |
| `git switch tarea-07-git` | 07 | **`07_git/`** |

La 08 nunca tuvo `07_git/`, así que esa carpeta desaparece al entrar y reaparece al volver a `tarea-07-git`. La tarea vive en su rama, en su fork y en su pull request.
:::

::: problem {#xg-2c title="Árbol 2c · Si se mergea la 07, ¿cambia la 08?"}
Se mergea la tarea 07. ¿Cambia lo que muestra el pull request de la 08?
:::

::: hint {of="xg-2c"}
¿Cargaba la 08 algún commit de la 07?
:::

::: answer {of="xg-2c"}
**No.** La base de la 08 sigue siendo el último `o`, y entre ese punto y E siguen estando sólo D y E.

Comprobado: después de mergear la 07 en `main`, `git log --oneline main..tarea-08-datacamp-intro` sigue listando sólo E y D.
:::

::: problem {#xg-2d title="Árbol 2d · ¿Por qué en el examen A sí cambiaba?"}
En el examen A del parcial, la 08 nació de la 07. ¿Por qué allá mergear la 07 sí cambiaba el pull request y aquí no?
:::

::: hint {of="xg-2d"}
Compara dónde nació la 08 en cada caso.
:::

::: answer {of="xg-2d"}
Allá la rama de la 08 **cargaba** A, B y C. Al mergear la 07, esos commits pasaban a la base y dejaban de aparecer. Aquí la 08 nunca cargó A y B, así que no hay nada que deje de aparecer.
:::

### Árbol 3 · Editó el archivo del curso

Carla no copió la plantilla a su carpeta: editó directo `codigo/docker/certificaciones.md` en su commit X. Mientras tanto el profesor cambió **esa misma línea** en el commit R.

```text
o---o---R          upstream/main
     \             (R cambia la línea 3)
      X            tarea-08-datacamp-intro
                   (X cambia la misma línea 3)
```

::: problem {#xg-3a title="Árbol 3a · ¿Qué dice la revisión automática?"}
¿Qué dice la revisión automática de su pull request, y por qué?
:::

::: hint {of="xg-3a"}
¿Qué regla de la revisión automática habla de la carpeta donde están los archivos?
:::

::: answer {of="xg-3a"}
**Rojo.** El pull request toca un archivo fuera de `estudiantes/carla/`. Si se mergeara, su cambio quedaría en la plantilla de todo el grupo.
:::

::: problem {#xg-3b title="Árbol 3b · ¿Merge limpio o conflicto?"}
Si alguien intentara mergearlo, ¿Git lo mezcla solo o hay conflicto? ¿Por qué?
:::

::: hint {of="xg-3b"}
¿Cuántos lados cambiaron la misma línea desde el ancestro común?
:::

::: answer {of="xg-3b"}
**Conflicto.**

Git compara las dos versiones contra el ancestro común, el último `o` antes de R:

| Línea 3 en… | Texto |
|---|---|
| El ancestro común | el original |
| R (curso) | **cambiada** |
| X (Carla) | **cambiada, distinto** |

Si sólo un lado hubiera cambiado la línea, Git se quedaría con ese. Aquí cambiaron **los dos**, y Git no puede saber cuál es la buena. Lo que contesta:

```text
Auto-merging codigo/docker/certificaciones.md
CONFLICT (content): Merge conflict in codigo/docker/certificaciones.md
Automatic merge failed; fix conflicts and then commit the result.
```
:::

::: problem {#xg-3c title="Árbol 3c · ¿Y si hubiera escrito en su copia?"}
Si hubiera escrito en `estudiantes/carla/docker/certificaciones.md`, ¿habría conflicto con R?
:::

::: hint {of="xg-3c"}
¿Es el mismo archivo, o son dos rutas distintas?
:::

::: answer {of="xg-3c"}
**No.** Son dos archivos en dos rutas distintas: R toca `codigo/docker/certificaciones.md` y X toca `estudiantes/carla/docker/certificaciones.md`. Ninguna línea la cambian los dos lados.

Comprobado: el merge termina solo, con `Merge made by the 'ort' strategy.`
:::

::: problem {#xg-3d title="Árbol 3d · ¿Qué regla lo evita?"}
¿Qué regla del curso evita esto, y por qué funciona aunque treinta personas entreguen la misma tarea?
:::

::: hint {of="xg-3d"}
Es la regla que dice dónde copias la plantilla y dónde trabajas.
:::

::: answer {of="xg-3d"}
**La regla del espejo**: copiar la plantilla a `estudiantes/<login>/` con la misma ruta, y trabajar sólo ahí. Cada quien escribe en una carpeta que nadie más toca, y nadie edita `codigo/`. Treinta pull requests no comparten ni una línea: ninguno choca con otro ni con las correcciones del curso.
:::

**Repasa:** [[branches-y-merge]], [[branches-en-serio]], [[el-flujo-del-curso]] y [[deshacer-en-git]].

## Parte 2 · Lee el error

Cada caso trae lo que la persona corrió y lo que Git contestó, tal cual. Escribe **qué pasó** y **cuál es el siguiente paso**.

::: problem {#xg-e1 title="Error 1 · Primer push de una rama nueva"}
```text
$ git push
fatal: The current branch tarea-09-uv-docker has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin tarea-09-uv-docker
```
:::

::: hint {of="xg-e1"}
«no upstream branch»: la rama nunca se ha subido. Git te sugiere el comando en el mismo mensaje.
:::

::: answer {of="xg-e1"}
**Qué pasó.** La rama existe en su máquina pero nunca se ha subido, y un `git push` a secas no sabe a dónde mandarla.

**Siguiente paso.** `git push -u origin tarea-09-uv-docker`. Con el `-u`, de ahí en adelante basta `git push`.
:::

::: problem {#xg-e2 title="Error 2 · Editó el README desde la página de GitHub"}
El día anterior editó el README de su fork **desde la página de GitHub**. Hoy, en su máquina:

```text
$ git push origin main
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:ana/fdd_o26_ana.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally.
```
:::

::: hint {of="xg-e2"}
«the remote contains work that you do not have locally»: ¿qué commit tiene el fork que tu máquina no?
:::

::: answer {of="xg-e2"}
**Qué pasó.** Su fork tiene un commit, la edición en la web, que su máquina no tiene. Git no sube para no borrarlo.

**Siguiente paso.** Traerlo, mezclarlo y volver a subir:

```bash
git fetch origin
git merge origin/main
git push origin main
```

Es el bloque A del ritual, con `origin` en lugar de `upstream`. `git pull origin main` hace lo mismo de un jalón (el curso configuró `pull.rebase false`). **Nunca** `--force`, porque borraría el commit de la web.
:::

::: problem {#xg-e3 title="Error 3 · Permiso denegado"}
Clonó el repositorio del curso y dejó para después la conexión con su fork.

```text
$ git push -u origin tarea-08-imagen
ERROR: Permission to raya-lucaria/fdd_o26.git denied to ana.
fatal: Could not read from remote repository.
```
:::

::: hint {of="xg-e3"}
¿A quién le intentaste subir? Lee el nombre del repositorio en el error.
:::

::: answer {of="xg-e3"}
**Qué pasó.** `origin` apunta al repositorio del curso, donde nadie del grupo puede escribir.

**Siguiente paso.** `git remote -v` para confirmarlo, y terminar de conectar: `git remote rename origin upstream` y `git remote add origin` con la URL de su fork.
:::

::: problem {#xg-e4 title="Error 4 · Cambiar de rama con trabajo sin guardar"}
Editó `notas.md` en la rama de la tarea, no hizo commit, y quiere volver a `main`.

```text
$ git switch main
error: Your local changes to the following files would be overwritten by checkout:
	estudiantes/ana/09_python/notas.md
Please commit your changes or stash them before you switch branches.
Aborting
```
:::

::: hint {of="xg-e4"}
Git se niega para no perder tu trabajo. ¿Cuáles son las dos formas de guardarlo antes de cambiar de rama?
:::

::: answer {of="xg-e4"}
**Qué pasó.** `notas.md` tiene un cambio sin guardar, y ese archivo es distinto en `main`. Cambiar de rama pondría en el disco la versión de `main` y borraría el cambio, así que Git se niega y no hace nada.

**Siguiente paso.** `git commit` si el cambio ya está listo. Si no, `git stash` y, de regreso en la rama, `git stash pop`.

Con `stash`, `notas.md` en cada zona (v1 = la versión del último commit de la rama, v2 = con la edición; Index es el staging area):

| Paso | Rama | Disco | Index | Último commit | Stash |
|---|---|---|---|---|---|
| Antes (el `switch` falla y no cambia nada) | tarea | **v2** | v1 | v1 | · |
| `git stash` | tarea | **v1** | v1 | v1 | **v2** |
| `git switch main`, y de regreso a la tarea | tarea | v1 | v1 | v1 | v2 |
| `git stash pop` | tarea | **v2** | v1 | v1 | **·** |

Si `notas.md` fuera idéntico en las dos ramas, `git switch main` no fallaría: se llevaría la edición a `main` y avisaría con `M …/notas.md` (comprobado).
:::

::: problem {#xg-e5 title="Error 5 · El .venv no aparece"}
Corrió `uv sync`, que creó `estudiantes/ana/09_python/uv_docker/.venv/` con cientos de archivos. Luego `git status --short` no muestra nada. ¿Se perdió el ambiente? ¿Es un problema?
:::

::: hint {of="xg-e5"}
¿Qué archivo del curso le dice a Git qué rutas no mirar?
:::

::: answer {of="xg-e5"}
**No se perdió, y no es un problema: así debe ser.** El `.gitignore` del curso tiene la línea `.venv`, así que Git no muestra esa carpeta. El ambiente sigue en el disco. No se sube porque se recrea con `uv sync`.

**El comando que lo prueba.** `git status --short --ignored` muestra también lo ignorado, marcado con `!!`:

```text
!! estudiantes/ana/09_python/uv_docker/.venv/
```
:::

::: problem {#xg-e6 title="Error 6 · Un .DS_Store apartado"}
Quiere sacar `.DS_Store` de lo que va a guardar **sin borrarlo** de su disco.

```text
$ git status
On branch tarea-09-uv-docker
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   estudiantes/ana/09_python/.DS_Store
	new file:   estudiantes/ana/09_python/uv_docker/pyproject.toml
```
:::

::: hint {of="xg-e6"}
El propio `git status` dice el comando entre paréntesis.
:::

::: answer {of="xg-e6"}
**`git restore --staged estudiantes/ana/09_python/.DS_Store`. Lo saca del Index (el staging area) y lo deja en el disco.**

| Paso | Disco | Index (nuevo, apartado) | Último commit |
|---|---|---|---|
| Antes | `.DS_Store`, `pyproject.toml` | `.DS_Store`, `pyproject.toml` | ninguno de los dos |
| `git restore --staged …/.DS_Store` | `.DS_Store`, `pyproject.toml` | **sólo `pyproject.toml`** | ninguno de los dos |

`.DS_Store` es basura que macOS crea en cada carpeta que abres en Finder. `git rm --cached` con la misma ruta hace lo mismo aquí (comprobado).

**Para que no vuelva a pasar.** El `.gitignore` del curso ya lista `.DS_Store`, y con esa línea `git add estudiantes/ana/09_python` no lo aparta. Si quedó apartado, su copia no tenía esa línea (un `main` sin actualizar) o se agregó con `git add -f`.
:::

::: problem {#xg-e7 title="Error 7 · Un commit que salió mal, sin push"}
Hizo commit y se dio cuenta de que le faltó un archivo y de que el mensaje está mal. **Todavía no hace push.** Quiere deshacer el commit pero conservar los cambios.
:::

::: hint {of="xg-e7"}
Necesitas deshacer el commit pero conservar los cambios apartados. En [[deshacer-en-git|Deshacer]] hay un `reset` que hace justo eso.
:::

::: answer {of="xg-e7"}
**`git reset --soft HEAD~1`. Deshace el commit y deja sus cambios apartados.**

| Paso | Disco | Index | Último commit |
|---|---|---|---|
| Después del commit malo | lo editado | igual al commit | **C**, mensaje malo, le falta un archivo |
| `git reset --soft HEAD~1` | igual | igual: lo de C sigue apartado | **el anterior a C** |
| `git add <lo que faltaba>` | igual | **lo de C + lo que faltaba** | el anterior a C |
| `git commit -m "<mensaje correcto>"` | igual | igual al commit | **commit nuevo, completo** |

`git reset --soft` no imprime nada. `git status --short` después muestra los archivos de C con la letra en la primera columna (`A` o `M`): siguen apartados (comprobado).

**Por qué sólo antes del push.** Reescribir un commit que ya subiste obliga a forzar el push, y eso borra historia en tu fork. Después del push, lo correcto es un commit nuevo que corrija.
:::

**Repasa:** [[deshacer-en-git]], [[lo-que-no-se-sube]] y la tabla de errores de [[cheatsheet-git]].

## Parte 3 · Encuentra los errores

Dani escribió este ritual para entregar `tarea-09-uv-docker`, cuya carpeta es `09_python/`. Lo corrió completo y **ningún comando falló**, pero la entrega salió mal. Ya tenía conectado su fork.

```text
 1   git switch -c tarea-09-uv-docker
 2   git switch main
 3   git fetch upstream
 4   git merge upstream/main
 5   git push origin main
 6   mkdir -p estudiantes/$GHUSER/09_python
 7   cp -r codigo/09_python estudiantes/$GHUSER/09_python/
 8   git status
 9   git add .
10   git commit -m "tarea 9"
11   git push -u origin main
```

::: problem {#xg-r1 title="a · Los cinco problemas"}
Hay **cinco** problemas: cuatro líneas mal y una que falta. Para cada uno, di la línea, qué se rompe y por qué.
:::

::: hint {of="xg-r1"}
Revisa cada línea contra el ritual del curso: el orden, el `cp`, el `add` y el `push`. Uno de los cinco es una línea que falta.
:::

::: answer {of="xg-r1"}
**Líneas 1, 7, 9 y 11 están mal, y falta un `git status` después de la 9.**

- **Línea 1.** Crea la rama antes de actualizar `main`, así que nace del `main` viejo. Además, la línea 2 saca a Dani de esa rama y la deja en `main`, y todo lo que sigue, commit y push incluidos, pasa en `main`. Arreglo: crear la rama **después** de la línea 5.
- **Línea 7.** `cp -r` copia **la carpeta entera**. Como el destino ya existe (lo creó la línea 6), la mete adentro: `09_python/09_python/`. Además se lleva `por_dentro/` y `stack/`, que son de otras entregas. El curso copia las dos carpetas por nombre: `cp -r codigo/09_python/ambientes codigo/09_python/uv_docker estudiantes/$GHUSER/09_python/`.
- **Línea 9.** `git add .` aparta todo lo que cambió en el repo, incluida la basura. El curso aparta por ruta, las dos carpetas por nombre: `git add estudiantes/$GHUSER/09_python/ambientes estudiantes/$GHUSER/09_python/uv_docker`.
- **Falta un `git status` después de la 9.** Es la última oportunidad de ver qué se va a guardar antes de guardarlo.
- **Línea 11.** Sube `main` y no la rama de la tarea. Desde `main` la revisión rechaza el pull request.
:::

::: problem {#xg-r2 title="b · ¿En qué rama quedaron los commits?"}
Después de correr todo, ¿en qué rama quedaron sus commits? ¿Qué rama subió a su fork?
:::

::: hint {of="xg-r2"}
Sigue con el dedo en qué rama estás después de cada `git switch`.
:::

::: answer {of="xg-r2"}
**En `main`. A su fork subió `main`, con su trabajo adentro. `tarea-09-uv-docker` existe, pero se quedó en el commit donde nació y nunca se subió.**

Supuesto: Dani empieza en `main`, y su `main`, el de su fork y `upstream/main` están en el mismo commit `o`. T es el commit de la línea 10.

| Línea | HEAD | `main` | `tarea-09-uv-docker` | `origin/main` | `upstream/main` |
|---|---|---|---|---|---|
| Inicio | `main` | o | · | o | o |
| 1. `git switch -c tarea-09-uv-docker` | **`tarea-09-uv-docker`** | o | **o** | o | o |
| 2. `git switch main` | **`main`** | o | o | o | o |
| 3 a 9 | `main` | o | o | o | o |
| 10. `git commit -m "tarea 9"` | `main` | **T** | o | o | o |
| 11. `git push -u origin main` | `main` | T | o | **T** | o |

Si el curso hubiera publicado algo antes, las líneas 3 y 4 moverían `upstream/main` y `main` a ese commit, y la conclusión sería la misma.

Al final:

```text
o---o          tarea-09-uv-docker (sin subir), upstream/main
     \
      T        main, origin/main
```

Es el árbol 1 de esta misma página. Comprobado: en el fork sólo existe `main`, y `git log --oneline upstream/main..tarea-09-uv-docker` no lista nada.
:::

::: problem {#xg-r3 title="c · ¿Dónde quedó hola.py?"}
Escribe la ruta donde quedó `hola.py`, que en el curso está en `codigo/09_python/ambientes/hola.py`.
:::

::: hint {of="xg-r3"}
¿Qué hace `cp -r` sin `/.` cuando la carpeta destino ya existe?
:::

::: answer {of="xg-r3"}
**`estudiantes/<login>/09_python/09_python/ambientes/hola.py`, con un `09_python` de más.**

Así no es espejo del curso, y la revisión no encuentra los archivos donde los busca.

Comprobado, con `GHUSER=dani`:

```text
$ find estudiantes -name hola.py
estudiantes/dani/09_python/09_python/ambientes/hola.py
```
:::

::: problem {#xg-r4 title="d · El ritual corregido"}
Escribe el ritual corregido, completo y en orden.
:::

::: hint {of="xg-r4"}
Es el ritual del curso con `tarea-09-uv-docker` y `09_python`. Empieza por `main`.
:::

::: answer {of="xg-r4"}
```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
git switch -c tarea-09-uv-docker
mkdir -p estudiantes/$GHUSER/09_python
cp -r codigo/09_python/ambientes codigo/09_python/uv_docker estudiantes/$GHUSER/09_python/
git status
git add estudiantes/$GHUSER/09_python/ambientes estudiantes/$GHUSER/09_python/uv_docker
git status
git commit -m "unidad 09: labs y mi ambiente uv en Docker"
git push -u origin tarea-09-uv-docker
```

Y al final el pull request en el navegador: base `raya-lucaria/fdd_o26` · `main`, compare `tarea-09-uv-docker`.
:::

**Repasa:** [[el-ritual-del-curso]].
