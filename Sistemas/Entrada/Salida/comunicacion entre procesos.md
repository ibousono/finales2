## Pipes

Los **pipes** son un mecanismo en sistemas Unix/Linux que permite la comunicación unidireccional entre procesos. **Se utilizan para transferir datos desde un proceso productor** (escribe datos) **hasta un proceso consumidor** (lee datos). Esto es especialmente útil para la sincronización y la transferencia de información en sistemas multitarea.

* **`int open(char* path, int flags, ...)`:** Abre el archivo indicado por
path, retornando un descriptor que apunta a dicho archivo.

* **`int close(int d):`** Cierra para el proceso actual el descriptor d
pasado por parámetro.

* **`int read(int d, void *b, size_t s)`:** Lee s bytes del archivo
apuntado por el descriptor d, y los escribe en el buffer b.

* **`int write(int d, void *b, size_t s):`** Lee s bytes del buffer b, y los
escribe en el archivo apuntado por el descriptor d.


### `int pipe(int descriptores[2])`
La función `pipe` crea un canal de comunicación unidireccional, llamado **pipe**, que tiene dos extremos:

1. **Extremo de escritura**: Es donde un proceso escribe datos.
2. **Extremo de lectura**: Es donde otro proceso puede leer esos datos.


#### Funcionamiento:
1. Cuando llamas a `pipe(int descriptores[2])`, el sistema operativo crea dos descriptores de archivo:
    * `descriptores[0]`: Representa el extremo de lectura.
    * `descriptores[1]`: Representa el extremo de escritura.


2. Estos descriptores se registran en la tabla de descriptores del proceso y se pueden usar con las syscalls `read` y `write`.


#### Pasos básicos:

#### 1. Crear un pipe:
```c
c

int descriptores[2];
if (pipe(descriptores) == -1) {
    perror("Error al crear el pipe");
    return 1;
}
```
#### 2. Escribir en el pipe (extremo de escritura):
```c
c

write(descriptores[1], "Hola", 4);
```
#### 3. Leer del pipe (extremo de lectura):
```c
c

char buffer[10];
read(descriptores[0], buffer, sizeof(buffer));
printf("Mensaje recibido: %s\n", buffer);
```

### Características importantes de los pipes:
#### Unidireccionales:
* Los datos solo pueden fluir en una dirección: del extremo de escritura al extremo de lectura.
#### Buffer interno:
* El kernel asigna un búfer para almacenar temporalmente los datos entre las operaciones de escritura y lectura.
#### Bloqueantes:
* Si el extremo de lectura no está listo, la escritura se bloquea hasta que alguien lea los datos (a menos que se configuren como no bloqueantes).

#### Ejemplo práctico de pipe con fork:
En este ejemplo, el padre escribe en el pipe y el hijo lee los datos.

```c
c

#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main() {
    int descriptores[2];
    pid_t pid;

    if (pipe(descriptores) == -1) {
        perror("Error al crear el pipe");
        return 1;
    }

    pid = fork();

    if (pid == -1) {
        perror("Error al crear el proceso hijo");
        return 1;
    }

    if (pid == 0) { // Proceso hijo
        close(descriptores[1]); // Cierra el extremo de escritura porque el hijo no necesita escribir
        char buffer[100];
        read(descriptores[0], buffer, sizeof(buffer));
        printf("Hijo recibió: %s\n", buffer);
        close(descriptores[0]); // Cierra el extremo de lectura
    } else { // Proceso padre
        close(descriptores[0]); // Cierra el extremo de lectura
        const char *mensaje = "Hola desde el padre";
        write(descriptores[1], mensaje, strlen(mensaje) + 1);
        close(descriptores[1]); // Cierra el extremo de escritura
    }

    return 0;
}
```

### `int dup2(int desc1, int desc2)`
La función `dup2` permite redirigir un descriptor de archivo existente a otro descriptor, es decir, el descriptor `desc2` apuntará al mismo recurso que `desc1`. Si `desc2` ya estaba asociado a otro recurso, esa asociación se elimina antes de la redirección.

### **¿Qué es un descriptor de archivo?**

Un descriptor de archivo es un número entero que el sistema operativo asigna a un recurso de entrada/salida (I/O). Estos recursos pueden ser:
- **Archivos** en el disco.
- **Entrada estándar** (teclado).
- **Salida estándar** (pantalla).
- **Sockets de red**.
- **Tuberías (pipes)**.
- Otros dispositivos.

El descriptor actúa como una "etiqueta" que permite a los programas identificar y operar sobre estos recursos. Ejemplos comunes:
- `0`: Entrada estándar (*stdin*).
- `1`: Salida estándar (*stdout*).
- `2`: Salida de error estándar (*stderr*).

Cuando usas `dup2`, lo que haces es **redirigir un descriptor existente** hacia otro, permitiendo que el segundo descriptor apunte al mismo recurso que el primero.

### Ejemplo básico con un archivo (recurso = archivo)
En este caso, el recurso involucrado es un archivo.

```c
c

#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main() {
    // Abrimos un archivo en modo escritura
    int archivo = open("salida.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (archivo == -1) {
        perror("Error al abrir el archivo");
        return 1;
    }

    // Redirigir stdout (1) al archivo
    dup2(archivo, STDOUT_FILENO);  // stdout ahora apunta al archivo
    close(archivo);  // Ya no necesitamos el descriptor original

    // Todo lo que se escriba en stdout irá al archivo
    printf("Esto va al archivo, no a la pantalla.\n");
    return 0;
}
```
1. **Recurso inicial**: El archivo `salida.txt`, que se abre con open.
2. **Asociación con stdout**: `dup2(archivo, STDOUT_FILENO)` redirige el descriptor `1` (stdout) al archivo.
3. **Resultado**: Todo lo que se escribe con `printf` (que usa stdout) se guarda en el archivo.


### Ejemplo con un pipe (recurso = pipe)
En este caso, el recurso es un pipe, y el objetivo es **redirigir la entrada estándar (stdin) para que lea del pipe**.

```c
c
Copiar
Editar
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main() {
    int descriptores[2];
    pipe(descriptores);  // Crear el pipe

    if (fork() == 0) {  // Proceso hijo
        close(descriptores[1]);  // Cerrar extremo de escritura
        dup2(descriptores[0], STDIN_FILENO);  // Redirigir stdin al extremo de lectura del pipe
        close(descriptores[0]);  // Ya no necesitamos este descriptor
        char buffer[100];
        read(STDIN_FILENO, buffer, sizeof(buffer));  // Leer del nuevo stdin
        printf("Hijo leyó: %s\n", buffer);
    } else {  // Proceso padre
        close(descriptores[0]);  // Cerrar extremo de lectura
        const char *mensaje = "Hola desde el padre";
        write(descriptores[1], mensaje, strlen(mensaje) + 1);  // Escribir al pipe
        close(descriptores[1]);  // Cerrar extremo de escritura
    }

    return 0;
}
```

1 .**Recurso inicial**: El pipe creado con `pipe(descriptores)`.

   * `descriptores[0]`: Extremo de lectura.
 * `descriptores[1]`: Extremo de escritura.
2. Asociación con stdin:

    En el proceso hijo, `dup2(descriptores[0], STDIN_FILENO)` redirige stdin     para que lea del pipe.
    

3 .Resultado:

* El hijo puede leer datos del pipe como si fueran de la entrada estándar.
