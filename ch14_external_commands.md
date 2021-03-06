# Capítulo 14: Comandos externos

Dentro del sistema Unix, encontrarás muchos comandos pequeños y muy específicos donde cada uno hace una tarea y la hace eficientemente. Puedes encadenar estos comandos para que funcionen de manera conjunta para dar solución a un problema complejo. ¿No sería genial si pudieras utilizar estos comandos dentro de Vim?

En este capítulo, aprenderás cómo extender las funcionalidades de Vim para que trabaje sin problemas con estos comandos externos.

## El comando *bang*

Vim tiene un comando llamado *bang* (`!`) que puede realizar tres cosas:

1. Leer la salida estándar (STDOUT) de un comando externo dentro del *buffer* actual.
2. Escribir el contenido de tu *buffer* como la entrada estándar (STDIN) a un comando externo.
3. Ejecutar un comando externo desde dentro de Vim.


## Leer la salida estándar (STDOUT) de un comando dentro de Vim

La sintaxis para leer la salida estándar (STDOUT) de un comando externo dentro del *buffer* actual es:

```
:r !{cmd}
```

`:r` es el comando de Vim para leer. Si lo utilizas sin `!`, puedes utilizarlo para obtener el contenido de un archivo. Si tienes un archivo llamado `archivo1.txt` en el directorio actual y ejecutas:

```
:r archivo1.txt
```

Vim pondrá el contenido de `archivo1.txt` dentro del *buffer* actual.

Si ejecutas el comado `:r` seguido por un `!` y un comando externo, la salida de ese comando será insertada dentro del *buffer* actual. Para obtener el resultado del comando `ls`, ejecuta:

```
:r !ls
```

Esto devolverá algo similar a esto:

```
archivo1.txt
archivo2.txt
archivo3.txt
```

Puedes leer los datos desde el comando `curl`:

```
:r !curl -s 'https://jsonplaceholder.typicode.com/todos/1'
```

El comando `r` también acepta una dirección:

```
:10r !cat archivo1.txt
```

Ahora la salida estándar (STDOUT) de ejecutar `cat archivo1.txt` será insertada después de la línea 10.

## Escribir el contenido de un *buffer* en un comando externo

Además de para guardar un archivo, también puedes utilizar el comando de escribir (`:w`) para pasar el contenido del *buffer* actual como la entrada estándar (STDIN) a un comando externo. La sintaxis es:

```
:w !cmd
```

Si tienes las siguientes expresiones en el *buffer*:

```
console.log("Hola Vim");
console.log("Vim es asombroso");
```

Asegúrate de tener [node](https://nodejs.org/en/) instalado en tu equipo y después ejecuta:

```
:w !node
```

Vim usará  `node` para ejecutar las expresiones de Javascript para mostrar por pantalla "Hola Vim" y "Vim is asombroso". 

Al utilizar el comando `:w`, Vim utiliza todo el texto del *buffer* actual, de manera similar al comando global (la mayoría de comandos de la línea de comandos, si no les pasas un rango, solo ejecutan el comando en la línea actual, no en todo el *buffer*). Si le pasas al comando `:w` una dirección específica:

```
:2w !node
```

Vim solo utilizará el texto de la segunda línea en el intérprete de `node`.

Existe una sutil pero significativa diferencia entre `:w !node` y `:w! node`. Con `:w !node`, estás "escribiendo" el texto del *buffer* actual en un comando externo, en este caso `node`. Con `:w! node`, estás forzando a guardar un archivo y dándole el nombre de "node".

## Ejecutando un comando externo

Puedes ejecutar un comando externo desde dentro de Vim con comando *bang*. La sintaxis es:

```
:!cmd
```

Verás el contenido del directorio actual en su formado extendido, ejecuta:

```
:!ls -ls
```

Para matar un proceso que se está ejecutando con el identificativo de proceso PID 3456, puedes ejecutar:

```
:!kill -9 3456
```

Puedes ejecutar cualquier comando externo sin dejar Vim y así permanecer centrado en tu tarea.

## Filtranto textos

Si das un rango al comando `!`, esto puede ser utilizado para filtrar textos. Supongamos que tenemos estos textos:

```
hello vim
hello vim
```

Vamos a convertir a mayúsculas la línea actual utilizando el comando `tr` (translate). Ejecuta:

```
:.!tr '[:lower:]' '[:upper:]'
```

Este es el resultado:

```
HELLO VIM
hello vim
```

Vamos a ver en detalle el comando:
- `.!` ejecuta el filtro del comando en la línea actual.
- `!tr '[:lower:]' '[:upper:]'` llama al comando `tr` para reemplazar los caracteres en minúsculas a mayúsculas.

Es imperativo pasar un rango para ejecutar el comando externo como un filtro. Si tratas de ejecutar el comando anterios sin el `.` (`:!tr '[:lower:]' '[:upper:]'`), verás un error.

Asumamos ahora que necesitamos eliminar la segunda columna de ambas líneas utilizando el comando `awk`:

```
:%!awk "{print $1}"
```

Este es el resultado:

```
hello
hello
```

La explicación:
- `:%!` ejecuta el filtro del comando en todas las líneas (`%`).
- `awk "{print $1}"` imprime solo la primera columna. En este caso la palabra "hello". 

Puedes encadenar múltiples comando con el operado `|` de igual manera que en la terminal. Supongamos que tenemos un archivo con estos elementos de un delicioso desayuno:

```
nombre precio
Tarde chocolate 10
Tarta merengue 9
Tarta frambuesa 12
```

Si necesitas ordenar la lisya en base al precio y mostrar solo el menu incluso con los espacios, puedes ejecutar:

```
:%!awk 'NR > 1' | sort -nk 3 | column -t
```

El resultado:
```
Tarta merengue 9
Tarde chocolate 10
Tarta frambuesa 12
```

La explicación:
- `:%!` aplica el filtro a todas las líneas (`%`).
- `awk 'NR > 1'` muestra los texto solo desde la columna 2 en adelante.
- `|` encadena esto con el siguiente comando.
- `sort -nk 3` ordena numéricamente (`n`) utilizando los valores de la columna 3 (`k 3`).
- `column -t` organiza el texto con los espaciados.

## Normal mode command

Vim has a filter operator (`!`) in the normal mode. If you have the following greetings:

```
hello vim
hola vim
bonjour vim
salve vim
```

To uppercase the current line and the line below, you can run:
```
!jtr '[a-z]' '[A-Z]'
```

The breakdown:
- `!j` runs the normal command filter operator (`!`) targetting the current line and the line below it. Recall that because it is a normal mode operator, the grammar rule `verb + noun` applies. 
- `tr '[a-z]' '[A-Z]'` replaces the lowercase letters with the uppercase letters.

The filter normal command only works on motions / text objects that are at least one line or longer. If you had tried running `!iwtr '[a-z]' '[A-Z]'` (execute `tr` on inner word), you will find that it applies the `tr` command on the entire line, not the word your cursor is on.

## Learn External Commands the Smart Way

Vim is not an IDE. It is a lightweight modal editor that is highly extensible by design. Because of this extensibility, you have easy access to any external command in your system. With this, Vim is one step closer from becoming an IDE. Someone said that the Unix system is the first IDE ever.

The bang command is as useful as how many external commands you know. Don't worry if your external command knowledge is limited. I still have a lot to learn too. Take this as a motivation for continuous learning. Whenever you need to filter a text, look if there is an external command that can solve your problem. Don't worry about mastering everything about a particular command. Just learn the ones you need to complete the current task.
