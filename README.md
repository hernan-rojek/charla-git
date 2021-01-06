# Introducción a Control de Versiones con Git

Vamos a ver el uso de Git desde abajo, así entendemos lo que estamos haciendo en vez de memorizar recetas que aplicamos cruzando los dedos y esperando que todo salga bien.

## Preliminares

Antes que eso, veamos cómo hacían hace unos cuarenta años para que varias personas pudieran trabajar sobre el mismo código.

A veces, almacenaban una copia en un repositorio central y ponían los archivos a ser modificados en modo *solo lectura*.

Otros equipos mantenían varias copias en paralelo y se comunicaban entre sí las partes que cambiaban. Vamos a profundizar en esta estrategia, porque es relevante para comprender el funcionamiento de Git.

### Parches

Supongamos que recibimos un diskette por correo con un proyecto para corregir. El proyecto se ve más o menos así: 

```console
dev0:corrector usuario$ tree -a -L 2 original/
original/
├── dias.txt
├── meses.txt
└── otros
    └── estaciones.txt

1 directory, 3 files
dev0:corrector usuario$
```

Consiste de unas listas de días, meses y estaciones. Contamos con poco tiempo para hacerlo, pero con un módem de algunos baudios, nos ahorraríamos el tiempo que insumiría devolver el diskette por correo. Si mandamos solamente las modificaciones, podemos ahorrar mucho tiempo de conexión telefónica. Por suerte, desde hace varias décadas existen las herramientas que permiten lograrlo.

Copiamos todo el árbol de directorios y hacemos las correcciones sobre la copia:

```console
dev0:corrector usuario$ cp -r original/ nuevo
dev0:corrector usuario$ cd nuevo/
dev0:nuevo usuario$ vim dias.txt
```

Notamos que falta un día. Lo agregamos:

<table>
  <tr>
    <th>original</th>
    <th>nuevo</th>
  </tr>
  <tr>
    <td>
      <pre>
domingo
lunes
martes
miércoles<br />
viernes
sábado
      </pre>
    </td>
    <td>
      <pre>
domingo
lunes
martes
miércoles
<b>jueves</b>
viernes
sábado
      </pre>
    </td>
  </tr>
</table>

Realizamos las correcciones sobre los meses:

```console
dev0:nuevo usuario$ vim meses.txt
```

<table>
  <tr>
    <th>original</th>
    <th>nuevo</th>
  </tr>
  <tr>
    <td>
      <pre>
enero
fe<b>rb</b>ero
marzo
abril
mayo
julio
agosto
<b>junio</b>
septiembre
octubre
noviembre
diciembre
      </pre>
    </td>
    <td>
      <pre>
enero
fe<b>br</b>ero
marzo
abril
mayo
<b>junio</b>
julio
agosto
septiembre
octubre
noviembre
diciembre
      </pre>
    </td>
  </tr>
</table>

Cambiamos el orden de las estaciones para que reflejen el del hemisferio sur:

```console
dev0:nuevo usuario$ vim otros/estaciones.txt
```
<table>
  <tr>
    <th>original</th>
    <th>nuevo</th>
  </tr>
  <tr>
    <td>
      <pre>
<b>invierno</b>
<b>primavera</b>
verano
otoño
      </pre>
    </td>
    <td>
      <pre>
verano
otoño
<b>invierno</b>
<b>primavera</b>
      </pre>
    </td>
  </tr>
</table>

Podemos ver las diferencias con el comando `diff`:

```console
dev0:nuevo usuario$ cd ..
dev0:corrector usuario$ diff -ruN original/ nuevo/
diff -ruN original/dias.txt nuevo/dias.txt
--- original/dias.txt   2020-10-01 15:51:09.819739706 -0300
+++ nuevo/dias.txt      2020-10-01 15:51:42.380478647 -0300
@@ -2,5 +2,6 @@
lunes
martes
miércoles
+jueves
viernes
sábado
diff -ruN original/meses.txt nuevo/meses.txt
--- original/meses.txt  2020-10-01 15:51:09.819739706 -0300
+++ nuevo/meses.txt     2020-10-01 15:51:55.260770956 -0300
@@ -1,11 +1,11 @@
enero
-ferbero
+febrero
marzo
abril
mayo
+junio
julio
agosto
-junio
septiembre
octubre
noviembre
diff -ruN original/otros/estaciones.txt nuevo/otros/estaciones.txt
--- original/otros/estaciones.txt       2020-10-01 15:51:09.819739706 -0300
+++ nuevo/otros/estaciones.txt  2020-10-01 15:52:05.661006981 -0300
@@ -1,4 +1,4 @@
-invierno
-primavera
verano
otoño
+invierno
+primavera
dev0:corrector usuario$
```
Estas son las instrucciones necesarias para corregir el proyecto.

Podemos almacenarlas en un archivo (típicamente denominado *parche*, *patch* o *delta*), y transferirlas al autor:

```console
dev0:corrector usuario$ diff -ruN original/ nuevo/ > correcciones.patch
dev0:corrector usuario$ cp correcciones.patch ../autor/
dev0:corrector usuario$ 
```
Nos impersonamos como el autor original:

```console
dev0:corrector usuario$ cd ../autor/
dev0:autor usuario$ ls
correcciones.patch original
dev0:autor usuario$ 
```
Y aplicamos las correcciones de la siguiente manera:

```console
dev0:autor usuario$ patch -p0 < correcciones.patch
patching file original/dias.txt
patching file original/meses.txt
patching file original/otros/estaciones.txt
dev0:autor usuario$ 
```

Podemos verificar que las versiones del autor y del corrector no tienen diferencias:

```console
dev0:autor usuario$ diff -ruN original/ ../corrector/nuevo/
dev0:autor usuario$ 
```

Antes de la aparición de los sistemas de control de versiones, este esquema de trabajo era común entre colaboradores ocasionales o geográficamente dispersos.
Cada desarrollador podía almacenar, junto con su copia de trabajo, el conjunto de deltas (diferencias) que lo llevaron a su estado actual.

La estrategia de operación de Git es básicamente eso mismo pero automatizado.

## Git

Git no es la única herramienta de control de versión, pero sí es la única que en la actualidad es considerada estándar de facto. Cualquier trabajo que un profesional del software emprenda hoy en día, casi con seguridad incluye interactuar con repositorios Git.

Al tratarse de una herramienta tan básica y omnipresente, entenderla desde lo fundamental es una inversión de alto retorno.

El modo de uso fundamental de Git es local. Un repositorio Git lleva a cuestas toda la historia del proyecto.
Esta característica es explícita en su diseño, ya que garantiza la performance necesaria para gestionar proyectos muy grandes.

La interacción del usuario siempre es con el repositorio local, mientras que la interacción con repositorios remotos se da siempre con el repositorio local mediante.
Esto es, un usuario no impacta cambios en un repositorio remoto de manera directa, sino que primero tiene que lograr un repositorio local en estado coherente, para luego hacerlo congeniar con repositorios remotos.

La rutina típica de bajarse una copia de un proyecto de un repositorio centralizado y contribuir cambios lleva implícito un workflow distribuído, por más que no lo parezca a primera vista.

Ciertas herramientas suelen contribuir a esta confusión: muchos agentes Git integrados en IDEs unifican interacciones locales con remotas, dando la apariencia de estar interactuando directamente con el repositorio remoto.
Muy posiblemente se deba a que previamente ese nicho funcional era ocupado por herramientas verdaderamente cliente/servidor, como CVS o SVN, y se haya buscado una transición lo más ergonómica posible.

Por este y otros motivos, recomendamos fuertemente fijar los conceptos propios de Git con la herramienta oficial, que funciona en terminales de línea de comandos como Bash o PowerShell.
Si resulta poco familiar o intimidante este tipo de entorno, es una buena oportunidad para interiorizarse un poco. Vamos a limitarnos a `git` y a comandos muy básicos, como por ejemplo:

  - `more`: muestra el contenido de un archivo
  - `ls`: lista el contenido de un directorio, si no se especifica, es del actual
  - `tree`: lo mismo que `ls`, pero mostrando el árbol en forma jerárquica 
  - `cd`: cambia el directorio actual
  - `mkdir`: crea un nuevo directorio

Adicionalmente, vamos a utilizar un editor de texto. En este caso usamos `vim`, pero como creemos que el masoquismo debe ser voluntario, donde usamos `vim`, se puede perfectamente usar el editor que quieran, como por ejemplo `nano`, que es un poco más amable.

### Configurando Git por primera vez

Git administra la configuración en tres niveles: por instalación, por usuario y por proyecto.
Esta configuración se almacena en archivos de texto, los cuales no es necesario modificar manualmente, ya que para este fin la herramienta provee el subcomando `git config`.

### Configuración por instalación

La configuración por instalación se almacena en `/etc/gitconfig` en sistemas UNIX o Linux, y en `C:\ProgramData\Git\config` en sistemas Windows. Para modificarla, el subcomando `git config` requiere la opción `--system`.

### Configuración por usuario

La configuración por usuario se almacena en `/home/[usuario]/.gitconfig` o bien en `/home/[usuario]/.config/git/config` en sistemas UNIX o Linux, y en `C:\Users\$USER` en sistemas Windows. Para modificarla, el subcomando `git config` requiere la opción `--global`.

### Configuración por proyecto

Ya dentro de un proyecto, la configuración es almacenada dentro del mismo repositorio. Esta se puede encontrar en  `.git/config`. Este es el caso por defecto del subcomando `git config`, para el cual existe la opción `--local` si se desea explicitarla.

Cabe destacar que si bien esta configuración es almacenada en el directorio ocupado por el repositorio, es propia de la instancia local, o sea, no se propaga a repositorios remotos.

### Ejemplo

La instalación de Git no incluye ninguna configuración inicial en ningún nivel. Podemos verificarlo para el nuevo usuario llamado convenientemente `usuario`:

```console
dev0:codigo usuario$ git config --list
dev0:codigo usuario$ 
```

Interactuar con un repositorio Git en este estado de precariedad no es imposible, ya que la herramienta puede deducir los datos requeridos para registrar los cambios a partir de la configuración del sistema operativo.
No obstante, se considera de buen gusto proveer nombre completo y dirección de correo electrónico estables, para evitar posteriormente tener que realizar modificaciones de bajo nivel al historial.

El nombre puede contener caracteres no-ASCII (por ejemplo los acentuados), pero éstos no son mostrados por defecto por Git en su historial, excepto que se setee la propiedad `core.quotepath off` en la configuración.

Debe tenerse en cuenta que si el proyecto es compartido con otros usuarios, éstos pueden no tener la misma configuración establecida (o no contar con el encoding adecuado), y verán los caracteres no-ASCII como códigos octales.
Estos problemas suelen abordarse con consenso sobre la configuración a utilizar, o bien decantando voluntariamente en un nombre condescendientemente anglocéntrico, como "Jose Maria Patinio".

Para nuestro usuario *usuario*, podemos establecer la configuración mínima de la siguiente manera:

```console
dev0:codigo usuario$ git config --global user.name "Usuario"
dev0:codigo usuario$ git config --list
user.name=Usuario
dev0:codigo usuario$ git config --global user.email "usuario@example.com"
dev0:codigo usuario$ git config --list
user.name=Usuario
user.email=usuario@example.com
dev0:codigo usuario$ 
```

Ésto se refleja en el archivo anteriormente mencionado:

```console
dev0:codigo usuario$ more ~/.gitconfig
[user]
  name = Usuario
  email = usuario@example.com
dev0:codigo usuario$ 
```

Git aplica la configuración a nivel sistema primero, luego la de usuario y finalmente la del proyecto, permitiendo de esta manera que el usuario tenga oportunidad de redefinir ciertas propiedades en proyectos específicos, como por ejemplo, un correo electrónico laboral en proyectos laborales y otro personal para el resto.

## El repositorio

Supongamos que nos gusta cocinar y mantener nuestras recetas debidamente documentadas. Para ello, hacemos uso de un ordenador personal y procedemos a crear un directorio para nuestro proyecto de recetario.

Usamos esta temática en lugar de código fuente para no mezclar terminologías similares y que quede bien clara la divisoria entre lo que es fuente y lo que es control de versión:

```console
dev0:codigo usuario$ mkdir recetario
dev0:codigo usuario$ cd recetario/
dev0:recetario usuario$ tree -a -L 1
.
0 directories, 0 files
dev0:recetario usuario$ 
```

Si contamos con la herramienta `git` [debidamente instalada](https://git-scm.com/downloads), podemos verificar que nuestro proyecto recetario no cuenta con un repositorio:

```console
dev0:recetario usuario$ git status
fatal: Not a git repository (or any of the parent directories): .git
dev0:recetario usuario$ 
```

Esto se remedia facilmente, inicializando un repositorio:

```console
dev0:recetario usuario$ git init
Initialized empty Git repository in /home/usuario/codigo/recetario/.git/
dev0:recetario usuario$ 
```

Con el subcomando `git init`, se inicializa el estado del repositorio local, almacenado en un directorio llamado `.git` dentro del directorio del proyecto (por convención UNIX, los archivos y directorios cuyos nombres comienzan con `.`, son considerados ocultos).
Profundizando un poco en la estructura de ese árbol de directorios, podemos apreciar que `git` guarda sus cosas ahí adentro:


```console
dev0:recetario usuario$ tree -a -L 2
.
└── .git
    ├── branches
    ├── config
    ├── description
    ├── HEAD
    ├── hooks
    ├── info
    ├── objects
    └── refs

6 directories, 3 files
dev0:recetario usuario$ 
```

Repetimos nuevamente el subcomando `git status`, y éste nos confirma que el repositorio ya existe:

```console
dev0:recetario usuario$ git status
On branch master

Initial commit

nothing to commit (create/copy files and use "git add" to track)
dev0:recetario usuario$ 
```

## Cambios


Comencemos por agregar una receta. Usando el procesador de texto que más nos guste, o el que consigamos:

```console
dev0:recetario usuario$ vim panqueques.txt
```

Podemos redactar una receta más o menos así. No es tan importante, necesitamos un ejemplo que pueda cambiar:

```
Panqueques (entre 9 y 12 unidades)

Ingredientes:
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

Preparación:
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente
```
<p align="right"> 
  <img width="288" height="288" src="wd.repo.panqueques.svg">
</p>

El directorio actual, sin considerar el subdirectorio `.git`, almacena lo que Git denomina *working copy*, o copia de trabajo.
Esta copia es sobre la cual trabajamos, y a partir de la cual Git computa las modificaciones que debe ir acumulando el historial.

Si volvemos a ejecutar el subcomando `git status`, la herramienta reconoce que hay un nuevo archivo del cual no guarda seguimiento:

```console
dev0:recetario usuario$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        panqueques.txt

nothing added to commit but untracked files present (use "git add" to track)
dev0:recetario usuario$ 
```

También podemos consultar el historial, el cual se encuentra vacío:

```console
dev0:recetario usuario$ git log
fatal: your current branch 'master' does not have any commits yet
dev0:recetario usuario$ 
```

Lo que nos está diciendo con eso es que la rama en la que estamos (se tomó el atrevimiento de ponerle *master*, pero eso se puede cambiar), no tiene ningún *commit*. Por *commit* se entiende un paquete de cambios que se realizan todos juntos, de manera atómica, sobre archivos y/o directorios preexistentes en la historia. Es similar al patch o delta que vimos preliminarmente, solo que además referencia a un commit anterior.

De esas cadenas de commits se deduce ese concepto de **rama** (*branch*), central en casi todo VCS como mecanismo para **aislar cambios**.

Cada commit es un fotograma en la historia de la fuente, y Git nos permite posicionarnos en cualquier punto de esta historia, mientras le indiquemos cual.
Mediante el subcomando `git add` ponemos a Git al corriente de que existe el archivo `panqueques.txt` agregándolo al índice del repositorio:

```console
dev0:recetario usuario$ git add panqueques.txt
dev0:recetario usuario$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   panqueques.txt

dev0:recetario usuario$ 
```

El índice (*index*) es una estructura de datos que determina en qué difiere la copia de trabajo con el fotograma utilizado como base. Agregar los cambios al índice de seguimiento se lo suele denominar *to stage* (algo así como "preparar", o "poner en escena"), ya que son los cambios que Git compone preliminarmente como nuevo paquete para crear un nuevo commit.

Como se puede apreciar ejecutando el subcomando `git log`, agregar un cambio al *stage* del repositorio, no altera la historia:

```console
dev0:recetario usuario$ git log
fatal: your current branch 'master' does not have any commits yet
dev0:recetario usuario$ 
```

Una vez que tenemos bien determinado el conjunto de cambios que va a componer el próximo commit, confirmamos la operación con el subcomando `git commit`:

```console
dev0:recetario usuario$ git commit -m "Nueva receta de panqueques"
[master (root-commit) 0f2d91b] Nueva receta de panqueques
 1 file changed, 9 insertions(+)
 create mode 100644 panqueques.txt
dev0:recetario usuario$ 
```

Este subcomando requiere un comentario en lenguaje de ser humano para poder facilitar la comprensión de la sucesión de cambios en el log del branch actual.
Luego de confirmar los últimos cambios, podemos ver que el índice no registra diferencias con respecto al último commit:

```console
dev0:recetario usuario$ git status
On branch master
nothing to commit, working tree clean
dev0:recetario usuario$ 
```

Estos cambios quedaron registrados en el historial:

```console
dev0:recetario usuario$ git --no-pager log
commit 0f2d91b99ba534624e6c7d007bf415bc3b2544e5
Author: Usuario <usuario@example.com>
Date:   Mon Sep 28 16:29:42 2020 -0300

    Nueva receta de panqueques
dev0:recetario usuario$ 
dev0:recetario usuario$ 
```
La opción `--no-pager` evita que el subcomando ejecute en modo pantalla completa, simplemente imprimiendo en la salida estándar. Si necesitamos algún detalle de esa salida para copiarlo textualmente en un comando subsiguiente, nos puede resultar útil tenerlo en la pantalla. Imaginen tener que memorizar ese `0f2d91b99ba534624e6c7d007bf415bc3b2544e5`.

Ese coso con números y letras es un número expresado en base hexadecimal, y se trata de un código de dispersión (*hash code*) derivado del contenido de los cambios y del código del commit anterior. Son números muy grandes, tanto que es virtualmente imposible encontrar dos commits diferentes con el mismo código de dispersión.

No hace falta entrar tanto en detalle sobre estos códigos, solamente necesitamos saber que son técnicamente únicos y permiten que dos colaboradores creen commits independientemente sin riesgo de que al intercambiarlos generen un conflicto.

Vamos a ilustrar una secuencia de cambios,

```console
dev0:recetario usuario$ vim panqueques.txt
```

agregando una línea nueva en el archivo:

<table>
  <tr>
    <th>original</th>
    <th>nuevo</th>
  </tr>
  <tr>
    <td>
      <pre>
Panqueques (entre 9 y 12 unidades)

Ingredientes:
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

Preparación:
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente<br />
      </pre>
    </td>
    <td>
      <pre>
Panqueques (entre 9 y 12 unidades)

Ingredientes:
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

Preparación:
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente
<b>dejar el panqueque reposando en una grilla</b>
      </pre>
    </td>
  </tr>
</table>

Vemos nuevamente que Git detecta el nuevo cambio:

```console
dev0:recetario usuario$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   panqueques.txt

no changes added to commit (use "git add" and/or "git commit -a")
dev0:recetario usuario$ 
```

Si olvidamos cuáles eran estos cambios, podemos invocar el subcomando `git diff`:

```console
dev0:recetario usuario$ git --no-pager diff
diff --git a/panqueques.txt b/panqueques.txt
index 8be4f7c..e077bcc 100644
--- a/panqueques.txt
+++ b/panqueques.txt
@@ -14,3 +14,4 @@ incorporar el azúcar o la sal
 incorporar la manteca derretida
 cocinar de ambos lados en sartén antiadherente
+dejar el panqueque reposando en una grilla
dev0:recetario usuario$ 
```

El subcomando `git diff` es similar al comando `diff` que vimos en la sección [Preliminares](#preliminares).
Sin parámetros, muestra la diferencia entre el directorio de trabajo y lo último commiteado al historial.

Agregamos el archivo, y ahora reconoce los cambios para ser commiteados:

```console
dev0:recetario usuario$ git add panqueques.txt
dev0:recetario usuario$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   panqueques.txt

```

Lo cual hacemos a continuación:

```console
dev0:recetario usuario$ git commit -m "Actualizo receta de panqueques"
[master 398e592] Actualizo receta de panqueques
 1 file changed, 1 insertion(+)
dev0:recetario usuario$ 
```

Pedir nuevamente el `log` (bitácora de la rama actual) nos muestra la cadena de commits, comenzando desde el más reciente:

```console
dev0:recetario usuario$ git --no-pager log
commit 398e59280c1bd44a6a50a6571bfa1c00974580f3
Author: Usuario <usuario@example.com>
Date:   Mon Sep 28 16:54:56 2020 -0300

  Actualizo receta de panqueques

commit 0f2d91b99ba534624e6c7d007bf415bc3b2544e5
Author: Usuario <usuario@example.com>
Date:   Mon Sep 28 16:29:42 2020 -0300

  Nueva receta de panqueques
dev0:recetario usuario$ 
```

Con esta nutrida historia de cambios ya podemos demostrar algunas de las posibilidades que habilita contar con control de versión.

Por ejemplo, podemos comparar el estado actual con el commit anterior:

```console
dev0:recetario usuario$ git --no-pager diff 0f2d91b
diff --git a/panqueques.txt b/panqueques.txt
index 8b134f1..8be4f7c 100644
--- a/panqueques.txt
+++ b/panqueques.txt
@@ -13,3 +13,4 @@ incorporar la harina
 incorporar el azúcar o la sal
 incorporar la manteca derretida
 cocinar de ambos lados en sartén antiadherente
+dejar el panqueque reposando en una grilla

```

El subcomando `git checkout` cambia el directorio de trabajo para que refleje los contenidos al momento del commit pasado como parámetro.

```console
dev0:recetario usuario$ git checkout 0f2d91b99ba534624e6c7d007bf415bc3b2544e5
Note: checking out '0f2d91b99ba534624e6c7d007bf415bc3b2544e5'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

  HEAD is now at 0f2d91b... 1
dev0:recetario usuario$ 
```

No es necesario pasar los 40 caracteres de código hash al comando. Git interpreta prefijos no ambiguos, de al menos cuatro caracteres.

El comando anterior podría haberse escrito: `git checkout 0f2d`.

El nuevo contenido del directorio de trabajo refleja el resultado de ese commit, pero Git advierte estar en estado de *detached HEAD*.

`HEAD` es una referencia que indica el punto de partida, o *base* del directorio de trabajo. Normalmente refiere simbólicamente a `refs/heads/master`, o sea, *el último commit del branch master*. 

Git se vale de esa referencia para saber qué cosas cambiaron y a partir de dónde, o dicho de otro modo, cuándo es que un directorio de trabajo se considera "limpio".

Al ejecutar `git checkout 0f2d`, Git computa el contenido del código fuente, lo reemplaza y reapunta la referencia `HEAD` a ese commit.
Esta advertencia es relevante porque el punto de partida referenciado por `HEAD` deja de ser móvil, con lo cual agregar nuevos commits equivale a trabajar con un branch sin nombre, solamente referenciable por el hash de su último commit.

Posicionar el `HEAD` en un commit específico es útil para consultar el estado de la fuente en ese instante o para crear una rama a partir de ese punto específico.

## Ramas

Regresemos el `HEAD` al branch `master`:

```console
dev0:recetario usuario$ git checkout master
Previous HEAD position was 0f2d91b... Nueva receta de panqueques
Switched to branch 'master'
dev0:recetario usuario$ 
```
Y creemos un branch para aislar algún conjunto de cambios, por ejemplo, la migración del recetario al formato Markdown.
De esta manera, podemos concentrarnos en migrar solamente lo escrito hasta el commit actual, mientras en el branch `master` seguimos agregando recetas nuevas o detalles a las existentes sin miedo a interferir con la migración.

```console
dev0:recetario usuario$ git branch migra-markdown
dev0:recetario usuario$ git checkout migra-markdown
Switched to branch 'migra-markdown'
dev0:recetario usuario$ 
```

Crear un nuevo branch no impacta sobre el historial, solamente se trata de la creación de una nueva referencia.

La ejecución posterior del subcomando `git checkout`, reapunta la referencia `HEAD` a la nueva referencia simbólica `refs/heads/migra-markdown`, o sea, una referencia móvil que cambia su valor a medida que se apilan commits.

<p align="center"> 
  <img width="288" height="288" src="wd.repo.panqueques.svg">
</p>

En esta nueva rama tenemos la tranquilidad de focalizarnos en cambiar de formato sin preocuparnos por los cambios en paralelo. Si la evolución que toma cada rama deviene en un conflicto, la resolución se toma en otra instancia.

```console
dev0:recetario usuario$ ls
panqueques.txt
dev0:recetario usuario$ git mv panqueques.txt panqueques.md
dev0:recetario usuario$ ls
panqueques.md
dev0:recetario usuario$ git status
On branch migra-markdown
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    panqueques.txt -> panqueques.md

dev0:recetario usuario$
```

El subcomando `git mv` es similar al comando básico `mv` de Bash, que mueve o renombra un archivo o directorio, con la diferencia de que en Git esa operación queda registrada de manera tal que se preserva la historia.

```console
dev0:recetario usuario$ git commit -m "Renombre txt a md"
[migra-markdown bedfafd] Renombre txt a md
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename panqueques.txt => panqueques.md (100%)
dev0:recetario usuario$ 
```

Comenzamos a cambiar el formato:

```console
dev0:recetario usuario$ vim panqueques.md
```

<table>
  <tr>
    <th>original</th>
    <th>nuevo</th>
  </tr>
  <tr>
    <td>
      <pre>
Panqueques (entre 9 y 12 unidades)

Ingredientes:
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

Preparación:
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente
dejar el panqueque reposando en una grilla
      </pre>
    </td>
    <td>
      <pre>
<b>&#35; Panqueques (entre 9 y 12 unidades)</b>

<b>&#35;&#35; Ingredientes:</b>
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

<b>&#35;&#35; Preparación:</b>
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente
dejar el panqueque reposando en una grilla
      </pre>
    </td>
  </tr>
</table>

```console
dev0:recetario usuario$ git add panqueques.md
dev0:recetario usuario$ git commit -m "Conversión a formato Markdown"
[migra-markdown bc7849d] Conversión a formato Markdown
 1 file changed, 16 insertions(+), 16 deletions(-)
 rewrite panqueques.md (99%)
dev0:recetario usuario$ 
```
Paralelamente, podemos evolucionar la receta en el branch `master`:

```console
dev0:recetario usuario$ git checkout master
Switched to branch 'master'
dev0:recetario usuario$ vim panqueques.txt
```
Notar que en este branch no renombramos el archivo.

<table>
  <tr>
    <th>original</th>
    <th>nuevo</th>
  </tr>
  <tr>
    <td>
      <pre>
Panqueques (entre 9 y 12 unidades)

Ingredientes:
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

Preparación:
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente
dejar el panqueque reposando en una grilla<br />
      </pre>
    </td>
    <td>
      <pre>
Panqueques (entre 9 y 12 unidades)

Ingredientes:
huevo, 1 unidad
leche, 1 taza
harina, 1 taza
azúcar o sal, a gusto
manteca derretida, una cucharada

Preparación:
batir el huevo y la leche
incorporar la harina
incorporar el azúcar o la sal
incorporar la manteca derretida
cocinar de ambos lados en sartén antiadherente
dejar el panqueque reposando en una grilla
<b>apilar los panqueques en un plato</b>
      </pre>
    </td>
  </tr>
</table>

```console
dev0:recetario usuario$ git add panqueques.txt 
dev0:recetario usuario$ git commit -m "Actualizo panqueques: apilar en el plato"
[master 35ad200] Actualizo panqueques: apilar en el plato
 1 file changed, 1 insertion(+)
dev0:recetario usuario$
```

Desde la perspectiva de `master`, el historial muestra los cambios relevantes:

```console
dev0:recetario usuario$ git --no-pager log --oneline
35ad200 Actualizo panqueques: apilar en el plato
398e592 Actualizo receta de panqueques
0f2d91b Nueva receta de panqueques
dev0:recetario usuario$
```
Para ver el historial completo, en orden cronológico, tenemos que agregar la opción `--all`:

```console
dev0:recetario usuario$ git --no-pager log --oneline --all
35ad200 Actualizo panqueques: apilar en el plato
bc7849d Conversión a formato Markdown
bedfafd Renombre txt a md
398e592 Actualizo receta de panqueques
0f2d91b Nueva receta de panqueques
dev0:recetario usuario$
```
Podemos además darle el contexto del lugar que ocupa cada commit en cada rama con la opción `--graph`:

```console
dev0:recetario usuario$ git --no-pager log --oneline --all --graph
* 35ad200 Actualizo panqueques: apilar en el plato
| * bc7849d Conversión a formato Markdown
| * bedfafd Renombre txt a md
|/
* 398e592 Actualizo receta de panqueques
* 0f2d91b Nueva receta de panqueques
dev0:recetario usuario$
```

## Merges

El procedimiento para incorporar los cambios del branch `migra-markdown` al branch `master` es básicamente así:

```console
dev0:recetario usuario$ git merge migra-markdown
```
En este momento, Git lanza un editor para componer un mensaje de commit. No es un mensaje común, sino que se supone que explica la naturaleza del merge.

Guardando el contenido y saliendo (con el editor `nano` se hace con <kbd>CTRL</kbd>+<kbd>O</kbd> seguido de <kbd>CTRL</kbd>+<kbd>X</kbd>), el merge se completa:

```console
dev0:recetario usuario$ git merge migra-markdown
Auto-merging panqueques.md
Merge made by the 'recursive' strategy.
 panqueques.txt => panqueques.md | 6 +++---
 1 file changed, 3 insertions(+), 3 deletions(-)
 rename panqueques.txt => panqueques.md (82%)
dev0:recetario usuario$ 
```

En el historial, el commit del merge queda así representado:

```console
dev0:recetario usuario$ git --no-pager log --oneline --all --graph 
*   a30d4e0 Merge branch 'migra-markdown'
|\
| * bc7849d Conversión a formato Markdown
| * bedfafd Renombre txt a md
* | 35ad200 Actualizo panqueques: apilar en el plato
|/
* 398e592 Actualizo receta de panqueques
* 0f2d91b Nueva receta de panqueques
dev0:recetario usuario$
```

En este caso de merge, Git cuenta con el contexto necesario para unificar los cambios de ambas ramas en el mismo archivo. No siempre es el caso, si Git no cuenta con líneas de texto estáticas en común para tomar como referencia, no puede deducir el orden para aplicar los cambios.

