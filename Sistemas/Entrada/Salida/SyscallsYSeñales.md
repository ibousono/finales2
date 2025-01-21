
## 1.  ¿Qué es una syscall?
Una **syscall** (abreviación de *system call* o llamada al sistema) es el mecanismo que permite a los programas de usuario interactuar con el kernel del sistema operativo. Es la única forma de acceder a los servicios esenciales que controla el kernel, como archivos, dispositivos, memoria, procesos, y recursos de red. Los programas no tienen acceso directo al hardware por razones de seguridad y estabilidad, por lo que las syscalls sirven como un "puente controlado".

### Funcionamiento básico de una syscall:

1. #### Transición del espacio de usuario al espacio del kernel:
    *    Los programas de usuario se ejecutan en el user space, mientras que el kernel opera en el *kernel space*. Las syscalls facilitan esta transición.


2. #### Ejecución de la operación en el kernel:
    * Una vez que el kernel recibe la syscall, ejecuta la tarea solicitada utilizando sus permisos de alto nivel.


3. #### Retorno al espacio de usuario:
    * El kernel devuelve el resultado al programa, que puede ser datos solicitados, el éxito de la operación o un código de error.

**Ejemplo práctico:**

* Un programa de usuario necesita escribir en un archivo. Esto implica la syscall `write`:


    1. El programa invoca la función `write` de la biblioteca estándar (`libc` en Linux).
    2. `libc` realiza la transición al kernel llamando a la syscall.
    3. El kernel utiliza su control sobre el sistema de archivos para realizar la escritura.
    4. Devuelve el número de bytes escritos o un código de error.
Ejemplo en C:

```c
c

#include <unistd.h>

int main() {
    const char *mensaje = "Hola, mundo!\n";
    write(1, mensaje, 13);  // Escribe en stdout
    return 0;
}
```
Aquí:

* `1` representa el descriptor de archivo para la salida estándar (*stdout*).
* `write` es una syscall que realiza la operación de escritura.




## 2. ¿Qué es un descriptor de archivo?
Un **descriptor de archivo** es un número entero que identifica un recurso de entrada/salida gestionado por el kernel. Puede representar un archivo, socket de red, dispositivo, o incluso memoria compartida. Este número permite que las syscalls realicen operaciones sobre ese recurso, como leer, escribir o cerrarlo. (Basicamente es un número que identifica un archivo o otras cosas)

### Cómo funcionan los descriptores:

1. #### Asignación:
    * Cuando un recurso es abierto mediante una syscall como `open` o `socket`, el kernel asigna el descriptor más bajo disponible.
1. #### Operaciones:
    * Las operaciones como `read`, `write` y `close` utilizan este descriptor para identificar el recurso.
1. #### Liberación:
    * Cuando el recurso ya no es necesario, el descriptor debe cerrarse con `close` para liberar recursos del sistema.
    * 
### Descriptores estándar en Unix/Linux:

* `0`: Entrada estándar (*stdin*).
* `1`: Salida estándar (*stdout*).
* `2`: Salida de error estándar (*stderr*).


**Ejemplo práctico en C:**

```c
c

#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("archivo.txt", O_WRONLY | O_CREAT, 0644);  // Abrir el archivo
    if (fd == -1) {
        return 1;  // Error al abrir el archivo
    }
    write(fd, "Hola, descriptor!\n", 18);  // Escribir en el archivo
    close(fd);  // Cerrar el archivo
    return 0;
}
```
**En este ejemplo:**

* `open` devuelve un descriptor para el archivo.
* `write` utiliza ese descriptor para escribir en el archivo.


## 3. ¿Qué es una API?
Una **API** (*Application Programming Interface*) es un conjunto de definiciones, funciones y protocolos que permite a los programas interactuar con sistemas, bibliotecas o servicios externos. Las APIs abstraen la complejidad subyacente, facilitando a los desarrolladores utilizar funcionalidades sin preocuparse por los detalles de implementación.

### Relación entre API y syscalls:

* Una API puede estar construida sobre syscalls, pero ofrece una interfaz más amigable para los desarrolladores.
* Ejemplo:
    * La función `fopen` de la biblioteca estándar de C utiliza internamente la syscall open.


**Ejemplo práctico:**

```c
c

#include <stdio.h>

int main() {
    FILE *archivo = fopen("archivo.txt", "w");  // Usar API fopen
    fprintf(archivo, "Hola desde una API!\n");  // Usar API fprintf
    fclose(archivo);
    return 0;
}
```
### Tipos de APIs:

1. #### APIs del sistema operativo:
    * Ejemplo: API de Linux que incluye funciones como `fork`, `exec`, y `wait`.
2. #### APIs web:
    * Ejemplo: APIs de Google Maps o OpenWeather para consultar servicios en línea.


## 4. ¿Cuando uso `echo`, estoy haciendo una syscall?
Cuando ejecutas `echo` en la terminal, como:

```bash
bash

echo "Hola, mundo"
```

indirectamente estás utilizando syscalls. `echo `es interpretado por el shell, que llama internamente a funciones que usan syscalls como `write`.

#### Cómo funciona `echo`:

1. El shell identifica que `echo` es un comando incorporado (builtin).
2. Utiliza funciones internas que eventualmente llaman a la syscall write para enviar texto a la salida estándar.


#### Visualización con strace:

```bash
bash

strace echo "Hola, mundo"
```
**Salida típica**:

``` c
plaintext

execve("/bin/echo", ["echo", "Hola, mundo"], ...) = 0
write(1, "Hola, mundo\n", 12) = 12
exit_group(0) = ?
```

## 5. ¿Qué es `execve` y cómo funciona?
La syscall `execve` es parte de la familia `exec` y se utiliza para reemplazar completamente la imagen del proceso actual con un nuevo programa. Esto significa que, cuando se llama a `execve`, el proceso actual deja de existir tal como lo conocemos, y su memoria se reemplaza con el contenido de un archivo ejecutable especificado.

`execve` es una syscall de bajo nivel, lo que significa que no realiza pasos adicionales como búsqueda en el` PATH`. Es más directa y cruda que otras funciones de la familia `exec`, como `execl` o `execvp`, que son envoltorios que facilitan su uso.

### Funcionamiento de execve:

#### 1. Validación del archivo ejecutable:

* El kernel verifica si el archivo especificado existe y tiene permisos de ejecución.
* Si no cumple estas condiciones, `execve` falla y devuelve `-1`.
#### 2.  Cierre de recursos innecesarios:

* El kernel descarta todo el contexto del proceso actual, incluyendo el código, la pila, y los segmentos de datos.
* Por defecto, los descriptores de archivo abiertos son heredados, a menos que estén configurados con la opción `close-on-exec`.
#### 3. Carga del programa en memoria:

* El kernel carga el nuevo programa desde el archivo ejecutable en la memoria del proceso.
* Configura los registros y los punteros del proceso para que apunten al nuevo programa.
####  4. Transferencia de control:

* La ejecución comienza desde el punto de entrada especificado en el ejecutable (normalmente, la función main en programas en C).


### Parámetros de execve:

La syscall execve tiene la siguiente firma:

```c
c

int execve(const char *pathname, char *const argv[], char *const envp[]);
```
#### 1. `pathname`:
 
* Es la ruta al archivo ejecutable que se va a cargar.
* Puede ser absoluta (por ejemplo,` /bin/ls`) o relativa al directorio actual.
#### 2. `argv[]`:
 
* Es una lista de argumentos que se pasarán al programa.
* `argv[0]` suele ser el nombre del programa por convención.
#### 3. `envp[]`:

* Es una lista de variables de entorno que se pasarán al nuevo programa.
* Si se pasa `NULL`, el programa heredará las variables de entorno del proceso padre.


**Ejemplo básico de execve:**

``` c
c

#include <unistd.h>

int main() {
    char *argv[] = {"/bin/ls", "-l", NULL};  // Argumentos del programa
    char *envp[] = {NULL};                  // Variables de entorno

    execve("/bin/ls", argv, envp);  // Reemplazar el proceso actual con `ls`
    return 0;  // Nunca se ejecutará si execve tiene éxito
}
```

**Explicación:**

**1.** El programa llama a `execve` con el ejecutable` /bin/ls`.
**2.** Se pasa el argumento `-l`, que indica que `ls` debe mostrar los archivos en formato largo.
**3.** Si `execve` tiene éxito, el proceso actual será reemplazado por el comando ls y mostrará la lista de archivos en el directorio actual.

---
### Errores comunes en execve:

* `ENOENT`: Archivo no encontrado.
* `EACCES`: Permiso denegado para ejecutar el archivo.
* `EINVAL`: Formato de archivo ejecutable inválido.
* `ENOMEM`: Memoria insuficiente para cargar el nuevo programa.

### Diferencia entre execve y otras funciones de la familia exec:

* `execl`: Usa argumentos pasados como lista en lugar de un array.
* `execvp`: Busca el ejecutable en los directorios del `PATH`.
* `execle`: Permite especificar variables de entorno.


## 6. ¿Cómo funciona `fork`?
La syscall `fork` crea un nuevo proceso duplicando el proceso actual. Es una de las syscalls más fundamentales en sistemas Unix/Linux y se utiliza para implementar multitarea.

El proceso recién creado se denomina **hijo** y es una copia casi exacta del proceso padre. Sin embargo, hay ciertas diferencias importantes que permiten a los dos procesos continuar ejecutándose independientemente.

### Funcionamiento de fork:

#### 1. Creación del proceso hijo:
 
* El kernel asigna un nuevo PID (identificador de proceso) para el hijo.
* Copia el espacio de memoria del padre para el hijo. Inicialmente, ambos comparten las mismas páginas de memoria, pero cualquier modificación en el contenido se realiza utilizando un mecanismo de *copy-on-write* (COW).
 
#### 2.  Diferenciación en el retorno:
 
 * En el proceso padre, `fork` devuelve el PID del proceso hijo.
 * En el proceso hijo, `fork` devuelve `0`.
 
#### 3.  Ejecución simultánea:

* Ambos procesos continúan ejecutándose a partir del punto donde se llamó a `fork`.

**Ejemplo básico de `fork`**:

```c
c

#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();  // Crear un proceso hijo

    if (pid == 0) {
        printf("Soy el hijo. Mi PID es %d\n", getpid());
    } else if (pid > 0) {
        printf("Soy el padre. El PID de mi hijo es %d\n", pid);
    } else {
        perror("Error al crear el proceso");
    }

    return 0;
}
```
**Explicación**:

1. El padre y el hijo imprimen diferentes mensajes basándose en el valor de retorno de fork.
2. Ambos procesos son independientes después de la creación.
Diferencias entre fork y vfork:

## 7. ¿Qué es wait y por qué es importante?
La syscall `wait` permite que un proceso padre espere a que uno de sus hijos termine. Esto asegura que el sistema operativo pueda liberar adecuadamente los recursos del hijo, evitando que quede en estado zombi.

#### Estado zombi:

* Cuando un hijo termina pero su padre no recoge su estado, el hijo permanece en la tabla de procesos como un zombi. Aunque no consume recursos de CPU, ocupa un espacio en la tabla de procesos.
Uso de wait:

```c
c
Copiar
Editar
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("Hijo ejecutándose\n");
    } else {
        int status;
        wait(&status);  // Esperar al hijo
        printf("El hijo terminó con estado %d\n", WEXITSTATUS(status));
    }

    return 0;
}
```
## 8. Señales (`signals`)
Una **señal** es un mecanismo que el kernel utiliza para notificar a los procesos sobre eventos o estados. Por ejemplo, se pueden enviar señales para interrumpir un proceso, terminarlo o informar sobre un error.

### Señales comunes:

* `SIGINT`: Interrupción (Ctrl+C).
* `SIGKILL`: Termina el proceso inmediatamente.
* `SIGCHLD`: Notifica al padre que un hijo cambió de estado.


**Ejemplo de manejo de señales**:

``` c
c

#include <signal.h>
#include <stdio.h>

void manejador(int sig) {
    printf("Recibí la señal %d\n", sig);
}

int main() {
    signal(SIGINT, manejador);  // Asociar SIGINT con el manejador
    while (1) {
        printf("Esperando señal...\n");
        sleep(1);
    }
    return 0;
}
```
Este programa captura `Ctrl+C` (SIGINT) y ejecuta el manejador en lugar de terminar.
