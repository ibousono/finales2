# Procesos
+ ¿Qué es una system call? ¿Para qué se usan? ¿Cómo funcionan? Explicar en detalle el funcionamiento una system call en particular

Una **system call** (llamada al sistema) es un mecanismo que permite a los programas de usuario solicitar servicios al sistema operativo. Estas llamadas son necesarias porque los programas de usuario no tienen acceso directo a los recursos del hardware o a las funciones privilegiadas del sistema. En su lugar, deben hacer una solicitud al kernel (núcleo del sistema operativo), que es el único con permisos para realizar estas operaciones.

### ¿Para qué se usan?
Las system calls se utilizan para realizar operaciones que requieren privilegios de kernel, como:

* **Crear o gestionar procesos** (por ejemplo, fork, exec, wait).

* **Gestionar archivos** (por ejemplo, open, read, write, close).

* **Comunicarse con otros procesos** (por ejemplo, pipe, shmget).

* **Solicitar recursos del sistema** (por ejemplo, malloc en última instancia usa brk o mmap).

Cuando un usuario invoca una syscall desde c, el proceso guarda los argumentos de la syscall en la pila, el harward guarda el PC y los registros, se pasa a la biblioteca, la biblioteca llama a trap para cambiar de modo usuario a modo kernell ( el sistema operativo guarda los datos necesarios del proceso actual en sus registros), el SO busca en la tabla o vector de system calls por el numero de syscall invocado y llama al handler con la rutina correspondiente y salta a ejecutarse a esa direccion

####  Vector de syscalls con las direcciones a sus handlers
![image](https://hackmd.io/_uploads/rJHhTY5tyl.png)

## ¿Para qué sirve la system call fork? ¿Qué debilidades tiene? Comparar con vfork y la creación de threads.

La system call **fork** se utiliza para crear un nuevo proceso, conocido como proceso hijo, que es una copia exacta del proceso que realiza la llamada, conocido como proceso padre. El proceso hijo hereda:

* El código ejecutable del padre.

* El espacio de memoria (una copia, gracias a Copy-On-Write).

* Los descriptores de archivos abiertos.

* El contexto de ejecución (registros, contador de programa, etc.).


Ejemplo de fork
![image](https://hackmd.io/_uploads/rJRNw_KKkg.png)

Cuando se hace el fork los dos procesos estan apuntando a lo mismo, mientras los dos solo lean va a seguir así, ahora si uno escribe el hijo carga una copia en otro lado de la memoria(copy on write)

Otro ejemplo
![image](https://hackmd.io/_uploads/rklUOdFYJg.png)

![image](https://hackmd.io/_uploads/HkeXOOYKkx.png)
 
 
 
 
Al  hacer un fork creas un proceso igual con el mismo codigo del padre, es como que hace una copia al codigo existente, si vos queres cargar un nuevo programa usas exec, que reemplaza la imagen actual(el programa actual) por una nueva imagen(un nuevo programa). Osea se descarta todo el espacio actual de memoria y se crea una nueva stack, heap todavia no hay porque no se pidió memoria, nueva seccion de codigo, basicamente todo lo que esta en el foto, pero nuevo, no lo del padre

**Debilidad**: Aunque fork usa **Copy-On-Write** para evitar copiar toda la memoria inmediatamente, aún puede ser costoso si el proceso padre es grande. `fork()` consume mucha más memoria que `vfork().`


En cambio `vfork` no realiza una copia del espacio de memoria del padre. En su lugar, el proceso hijo comparte el espacio de memoria del padre hasta que llama a `exec` o `exit`. Esto hace que `vfork` sea más rápido y eficiente en términos de memoria.

El proceso hijo **no** debe modificar el espacio de memoria compartido antes de llamar a `exec` o `exit`.

El proceso padre se bloquea hasta que el hijo llama a `exec` o `exit`.

Por otro lado crear un proceso con `fork` es más costoso que crear un `thread`, ya que implica copiar el espacio de memoria.

Los `threads` son más ligeros y rápidos de crear.

## Diferencias entre system calls para crear procesos entre Linux y Windows

En Linux, la creación de procesos se basa en la system call fork y sus variantes, seguidas de exec para cargar un nuevo programa.

En Windows, la creación de procesos es más integrada: CreateProcess hace todo en una sola llamada.


## ¿Para qué sirve la paginación de la memoria? ¿Qué ventajas tiene sobre utilizar direcciones físicas? Hablar sobre el tamaño de las páginas y cómo se relaciona con el tamaño de los bloques en disco. (hablar de fragmentación interna y fragmentación externa)

Paginación de la memoria sirve para gestionar la memoria de manera eficiente, permitir la memoria virtual, aislar procesos y facilitar la compartición de memoria.

Ventajas sobre direcciones físicas: Abstracción, eficiencia en la asignación, soporte para memoria virtual, y protección.

Tamaño de las páginas: Compensación entre overhead de gestión y eficiencia de E/S. Suele ser de 4 KB, pero puede variar.

## ¿Por qué puede pasar que tengamos muchos procesos en waiting, y cómo podría tratar de arreglarlo si no pudiese agregar memoria?

Espera por E/S:

Si los procesos están realizando muchas operaciones de entrada/salida (E/S), pueden pasar mucho tiempo en estado waiting mientras esperan que se completen esas operaciones.

Deadlocks:

Un deadlock ocurre cuando dos o más procesos están esperando indefinidamente por recursos que están siendo retenidos por otros procesos en espera.

Ejemplo: El proceso A tiene el recurso X y espera por el recurso Y, mientras que el proceso B tiene el recurso Y y espera por el recurso X.

Falta de memoria (thrashing):

Si el sistema no tiene suficiente memoria física, puede ocurrir thrashing, donde el sistema pasa más tiempo intercambiando páginas entre la RAM y el disco que ejecutando procesos.

Esto hace que los procesos pasen mucho tiempo en estado waiting mientras sus páginas son cargadas desde el disco.

solución: 

Reducir el número de procesos, talvez mataandolos o poniendolos en uno solo


## Hablar de RAID (para qué sirve). Explicar la diferencia entre RAID 4 y RAID 5. ¿Cuál es mejor y por qué?

RAID (Redundant Array of Independent Disks) es una tecnología que combina múltiples discos duros en una sola unidad lógica para mejorar el rendimiento, la fiabilidad, o ambas cosas.

Hay dos tipos de configuraciones

**mirroring**: donde se busca la redundancia de los datos
**stripping**: donde se busca conseguir mayor velocidad de transferencia de datos

#### RAID 1: Mirroring (Duplicación)

Los datos se duplican en dos o más discos.
Cada disco contiene una copia exacta de los datos.

Ventajas:
- Si un disco falla, los datos están disponibles en el otro(s).
- Buena velocidad de lectura

Desventajas:
- Se necesita el doble (o más) de espacio de almacenamiento.
- Velocidad de escritura limitada

#### RAID 2: Bit-level Striping con Paridad (Obsoleto)

Los datos se dividen a nivel de bits y se distribuyen entre los discos.

Se utilizan discos adicionales para almacenar códigos de corrección de errores (ECC).

Ventajas:
- Detección y corrección de errores: Gracias a los discos de ECC.

Desventajas:
- Complejidad: Requiere hardware especializado para manejar los códigos de corrección.

- Ineficiente: No es práctico para la mayoría de las aplicaciones modernas.

#### RAID 3: Byte-level Striping con Paridad Dedicada

Los datos se dividen a nivel de bytes y se distribuyen entre los discos.

Se utiliza un disco dedicado para almacenar la información de paridad.

Ventajas:

- Buena velocidad de lectura: Los datos se distribuyen entre varios discos.

- Redundancia: Puede tolerar la falla de un disco.

Desventajas:
- Cuello de botella en escrituras: El disco de paridad dedicado puede limitar el rendimiento de escritura.

- Complejidad: Menos común que RAID 5.

#### RAID 4: Block-level Striping con Paridad Dedicada

Los datos se dividen en bloques y se distribuyen entre los discos.

Se utiliza un disco dedicado para almacenar la información de paridad.

Ventajas:
- Buena velocidad de lectura: Los datos se distribuyen entre varios discos.

- Redundancia: Puede tolerar la falla de un disco.

Desventajas:
- Cuello de botella en escrituras: El disco de paridad dedicado puede limitar el rendimiento de escritura.

- Menos eficiente que RAID 5: La paridad dedicada es menos escalable.

#### RAID 5: Block-level Striping con Paridad Distribuida

Los datos se dividen en bloques y se distribuyen entre los discos.

La paridad se distribuye entre todos los discos, en lugar de estar en un disco dedicado.

Ventajas:
- Redundancia: Puede tolerar la falla de un disco.

- Buena velocidad de lectura: Los datos se distribuyen entre varios discos.

- Sin cuello de botella en escrituras: La paridad distribuida evita el cuello de botella de RAID 4.

Desventajas:
- Rendimiento de escritura: El cálculo de la paridad puede ralentizar las escrituras.

- Complejidad: Más complejo de implementar que RAID 1 o RAID 0.


## Explicar los distintos algoritmos básicos de scheduling de disco. Explicar cómo se comportan en HDD y en SDD.

#### FCFS (First-Come, First-Served):

Cómo funciona: Atiende las solicitudes en el orden en que llegan.

Ventajas: Simple y justo.

Desventajas: Puede resultar ineficiente en términos de movimiento del cabezal del disco, especialmente si las solicitudes están muy dispersas.

#### SSTF (Shortest Seek Time First):

Cómo funciona: Selecciona la solicitud más cercana a la posición actual del cabezal.

Ventajas: Reduce el tiempo de búsqueda (seek time) y mejora el rendimiento en HDD.

Desventajas: Puede causar inanición (starvation) para solicitudes que están lejos de la posición actual.

#### SCAN (Elevator Algorithm):

Cómo funciona: El cabezal se mueve en una dirección (por ejemplo, de adentro hacia afuera), atendiendo las solicitudes en su camino, y luego invierte la dirección.

Ventajas: Equilibra el rendimiento y la justicia.

Desventajas: Las solicitudes en los extremos del disco pueden experimentar mayores tiempos de espera.

#### C-SCAN (Circular SCAN):

Cómo funciona: Similar a SCAN, pero cuando el cabezal llega al final del disco, regresa al inicio sin atender solicitudes en el camino de retorno.

Ventajas: Más justo que SCAN, ya que reduce la variabilidad en los tiempos de espera.

Desventajas: Puede ser menos eficiente que SCAN en términos de movimiento total del cabezal.

#### LOOK y C-LOOK:

Cómo funciona: Versiones optimizadas de SCAN y C-SCAN que no llegan necesariamente al final del disco, sino que cambian de dirección cuando no hay más solicitudes en la dirección actual.

Ventajas: Más eficientes que SCAN y C-SCAN, ya que reducen el movimiento innecesario del cabezal.

Desventajas: Similar a SCAN y C-SCAN, pero con menos movimiento.

### Comportamiento en HDD (Discos Duros Mecánicos) y SSD (Discos de Estado Sólido)

En los HDD, el tiempo de acceso a los datos está dominado por:

* Tiempo de búsqueda (seek time): El tiempo que tarda el cabezal en moverse a la pista correcta.

* Latencia rotacional: El tiempo que tarda el disco en girar hasta que el sector deseado está bajo el cabezal.

En los SSD, no hay partes móviles, por lo que:

* No hay tiempo de búsqueda ni latencia rotacional: El acceso a los datos es casi instantáneo, independientemente de su ubicación física.

* El rendimiento está más relacionado con la latencia de las celdas de memoria y el controlador.

| Algoritmo | Comportamiento en HDD | Comportamiento en SSD |
|-----------|-----------------------|-----------------------|
| **FCFS**  | Ineficiente si las solicitudes están dispersas | Funciona bien, sin penalización por movimiento |
| **SSTF**  | Reduce el tiempo de búsqueda, pero puede causar inanición | No es necesario, ya que no hay tiempo de búsqueda |
| **SCAN**  | Equilibra rendimiento y justicia | No es relevante, ya que no hay movimiento físico |
| **C-SCAN**| Más justo que SCAN, pero con más movimiento | No es relevante |
| **LOOK/C-LOOK** | Optimiza el movimiento del cabezal | No es relevante |

## ¿Qué son los drivers y qué permiten?

Los **drivers** (o controladores) son programas especializados que **permiten que el sistema operativo se comunique con el hardware de una computadora**. Actúan como intermediarios entre el sistema operativo y los dispositivos físicos, como impresoras, tarjetas gráficas, discos duros, teclados

Corren con máximo privilegio: pueden hacer colgarse a todo elsistema.De ellos depende el rendimiento de E/S, que es fundamental para el rendimiento combinado del sistema.
