# CTF - Esteganografía en GIF

Tres desafíos, tres niveles de dificultad y un mismo objetivo: encontrar la flag escondida.

Este repositorio contiene una serie de desafíos de esteganografía pensados para aprender, paso a paso, qué hacer cuando te entregan un archivo en un CTF de forense. Cada nivel esconde la flag con una técnica distinta, y todas se resuelven con herramientas que ya vienen incluidas en Kali Linux.

El formato de la flag es:

```
FLAG{...}
```

## Qué es la esteganografía

Es la técnica de ocultar información dentro de archivos (imágenes, audio, video) sin que se note a simple vista. A diferencia de la criptografía, que cifra el mensaje, la esteganografía esconde su existencia.

Mentalidad CTF: cuando te dan un archivo, la pregunta no es "¿tiene algo oculto?", eso asúmelo como un sí. La pregunta real es: ¿dónde y cómo está oculto?

Para responderla, hazte siempre estas preguntas:

1. ¿Qué tipo de archivo es realmente?
2. ¿Tiene más datos de lo esperado?
3. ¿Sus metadatos dicen algo inusual?
4. ¿Hay archivos embebidos dentro?

## Los desafíos

Los tres desafíos usan un archivo `rickroll.gif`. Los tres se ven exactamente igual al abrirlos; la diferencia está en lo que no se ve.

**Nivel fácil** (`nivel_facil/rickroll.gif`)
Un GIF aparentemente normal. La información no está en la imagen, sino después de ella. Es el mejor punto de partida para aprender a mirar un archivo más allá de lo que muestra el visor.

**Nivel medio** (`nivel_medio/rickroll.gif`)
Otro GIF, pero esta vez el secreto se esconde en los datos internos del archivo y viene con una capa extra de ofuscación, por lo que encontrarlo no es suficiente: también hay que lograr leerlo.

**Nivel difícil** (`nivel_dificil/rickroll.gif`)
Un GIF que guarda otro archivo dentro de sí, y ese archivo está protegido. Tendrás que detectarlo, extraerlo y encontrar la forma de abrirlo.

Cada nivel agrega una capa más que el anterior: el fácil requiere una técnica, el medio una técnica más una decodificación, y el difícil combina extracción, obtención de una contraseña y descompresión.

## Herramientas y comandos

### Checklist estándar

Cuando recibas un archivo en un CTF de forense o esteganografía, ejecuta esto siempre, en orden:

```bash
file archivo.gif
strings archivo.gif | grep -i "flag"
exiftool archivo.gif
xxd archivo.gif | head -20 && xxd archivo.gif | tail -20
binwalk archivo.gif
```

A continuación se explica qué hace cada herramienta.

### file

Identifica el tipo real de un archivo mirando su contenido, no su extensión. Un archivo puede llamarse `.gif` y ser otra cosa.

```bash
file archivo.gif
```

### strings

Recorre un archivo binario y muestra todas las cadenas de texto legibles que encuentra. Combinado con `grep` permite filtrar y buscar una palabra concreta.

```bash
strings archivo.gif                      # muestra todo el texto legible
strings archivo.gif | grep -i "flag"     # filtra solo las líneas que contienen "flag"
```

- `|` (pipe): pasa la salida de `strings` como entrada de `grep`.
- `grep -i`: busca un texto ignorando mayúsculas y minúsculas.

### xxd

Muestra el contenido de un archivo en hexadecimal (columna izquierda) junto con su equivalente en texto ASCII (columna derecha). Permite ver los bytes reales del archivo, incluyendo cabeceras y datos al final.

```bash
xxd archivo.gif
```

Los formatos de archivo tienen marcadores que indican dónde empiezan y dónde terminan. Si sabes cuál es el marcador de fin de un GIF, sabrás dónde mirar.

### head y tail

Muestran las primeras o las últimas líneas de una salida. Como `xxd` produce muchísimas líneas, se usan para ver solo el inicio o el final del archivo.

```bash
xxd archivo.gif | head -20     # primeras 20 líneas
xxd archivo.gif | tail -20     # últimas 20 líneas
```

- `-20`: cantidad de líneas a mostrar. Puedes cambiarla según lo que necesites.

### exiftool

Muestra los metadatos de un archivo: información que viaja dentro de él pero que no se ve al abrirlo (fecha, autor, software, comentarios, GPS, etc.).

```bash
exiftool archivo.gif
```

Revisa con atención los campos que no te parezcan normales.

### binwalk

Analiza un archivo buscando firmas de otros formatos escondidos dentro de él (ZIP, imágenes, ejecutables, etc.) e indica en qué posición (offset) empiezan.

```bash
binwalk archivo.gif        # solo detecta y lista
binwalk -e archivo.gif     # detecta y extrae lo encontrado
```

- `-e`: extract. Extrae automáticamente los archivos que encuentre. Los deja en una carpeta nueva llamada `_archivo.gif.extracted/`.

### unzip

Descomprime archivos `.zip`. Lo necesitarás para abrir los `.zip` que extraigas de los desafíos.

```bash
unzip archivo.zip                    # descomprime
unzip -P contraseña archivo.zip      # descomprime un ZIP protegido con contraseña
unzip -l archivo.zip                 # lista el contenido sin extraerlo
```

- `-P`: indica la contraseña del ZIP.
- `-l`: list. Muestra qué hay dentro sin extraerlo.

### gunzip

Descomprime archivos `.gz` (gzip). Se usa para descomprimir la wordlist rockyou, que en Kali viene comprimida.

```bash
# Ver dónde están las wordlists en Kali
ls /usr/share/wordlists/

# Descomprimir rockyou (requiere sudo)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Después de esto tendrás el archivo `/usr/share/wordlists/rockyou.txt`.

Si prefieres conservar también el `.gz` original, usa la opción `-k`:

```bash
sudo gunzip -k /usr/share/wordlists/rockyou.txt.gz
```

### rockyou.txt

Es una lista con millones de contraseñas comunes filtradas históricamente. En CTFs se usa como diccionario para probar contraseñas por fuerza bruta. Una herramienta que la utiliza es `fcrackzip`.

## Otras herramientas que puedes necesitar

- `base64`: codifica y decodifica texto en Base64. Ejemplo: `echo "texto" | base64 -d`
- `fcrackzip`: fuerza bruta de contraseñas de archivos ZIP usando una wordlist.

## Cómo empezar

1. Abre una terminal en Kali Linux.
2. Entra a la carpeta del nivel que quieras resolver:
   ```bash
   cd nivel_facil/
   ```
3. Ejecuta el checklist estándar sobre `rickroll.gif`.
4. Analiza los resultados e identifica qué te llama la atención.
5. Cuando encuentres la flag, verifica que tenga el formato `FLAG{...}`.

Se recomienda avanzar en orden: fácil, medio y difícil. Cada nivel enseña una técnica que se reutiliza en el siguiente.

## Resumen de herramientas por nivel

- Todos los niveles: `file`, `xxd`, `head` y `tail`.
- Nivel fácil: `strings`.
- Nivel medio: `exiftool` y `base64`.
- Nivel difícil: `binwalk`, `unzip`, `gunzip` y `fcrackzip`.
