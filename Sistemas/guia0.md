# guia 0

## ej 1


![image](https://github.com/user-attachments/assets/0bfe8a39-c037-4f55-9876-1bf7c002b75b)


### **a) ¿Qué hace el comando "whoami"?**

> Printea el user id

### **b) ¿Qué hace el comando "uname"? ¿Para qué sirve la opción "-a"?**

> Printea informacion del sistema
> 
> Por ejemplo `uname -a` printea toda la info del sistema
> 
> `uname -o` printea el sistema oprativo

### **c) ¿Qué hace el comando "id"?**

> printea el user id

### **d) ¿Qué hace el comando "ps"? ¿Para qué sirve la opción ""-e"?**

> printea un snapshot de todos los procesos actuales

 `ps -e ` printea todos los procesos del sistema

### **e) ¿Qué hace el comando "top"? ¿Qué sucede al ejecutar "top -n 10"?**

> Te printea todos los procesos y su informacion `provides  a  dynamic real-time view of a running system`
> 
> top -n 10 se especifica el parámetro -n para indicar el número de actualizaciones que se deben realizar antes de que el comando termine. En este caso, al ejecutar top -n 10, el comando:

1.     Muestra la información sobre los procesos.
2.     Actualiza la información 10 veces.
3.     Después de la décima actualización, finaliza y sale automáticamente
    
    
    
### **f) ¿Qué hace el comando "pwd"?**


> printea la ruta en la que estas parado

### **g) ¿Qué hace el comando "ls"?**
 
> ls lista los contenidos de donde estas parado(archivos, directorios, etc)

### **h) ¿Qué hace el comando "mkdir"?**

> crea un nuevo directorio ej: `mkdir hola`


### **i) ¿Qué hace el comando "touch"?**

> crea un nuevo archivo ej: `touch hola.txt`


### **j) ¿Qué hace el comando "rm"?** 

> borra archivos(no directorios) 

### **k) ¿Qué hace el comando "rm -r"?**

> borra directorios ej `rm -r hola`

## Segunda parte

![image](https://github.com/user-attachments/assets/33c57c04-6d1c-4319-b2ee-1480af50be55)



![image](https://github.com/user-attachments/assets/33a7bcb7-0bea-4178-95d3-5018c2697b65)



![image](https://github.com/user-attachments/assets/fb30dcf9-da3e-458c-8d39-c357a7639161)

### Tercera parte

El sufijo **.sh** en una archivo(ej: hola.sh) es una convención para identificar que el archivo es un **bash script**.


Un **Bash script** es un archivo de texto que contiene una serie de comandos que se pueden ejecutar de manera secuencial en una terminal de Linux, o en sistemas basados en Unix. **Bash** (Bourne Again Shell) es un *intérprete de comandos* que ejecuta estos scripts. 

Un **intérprete de comandos** (o shell) es un programa que permite a los usuarios interactuar con el sistema operativo a través de comandos de texto. **Interpreta** y **ejecuta** los comandos que introduces en la terminal. Cuando escribes un comando, el shell es el que toma ese comando, lo interpreta y luego ejecuta la acción correspondiente.Actúa como una interfaz entre el usuario y el núcleo del sistema operativo, interpretando las órdenes que el usuario escribe y ejecutándolas.

La **terminal** y el **intérprete de comandos** no son lo mismo, aunque están estrechamente relacionados y suelen trabajar juntos.





---


---


---


### a) **¿Qué hace el comando "cat" o "more"?**

> Muestran el contenido de un archivo ej:  `cat hola.sh, more hola.sh`

### b)**¿Qué hace el comando "grep"?**

> permite especificar un patrón y buscar las líneas del archivo que coincidan con el patrón ej: `grep escalera hola.sh` (*impreme en la terminal todas las lineas de hola.sh con la palabra escalera*)

### **d) Como se usa un bash script?**

> Creas el archivo con touch script.sh y le pones lo siguiente:

![imagen](https://github.com/user-attachments/assets/558ba716-07fc-4d4e-97aa-76bab41ea29e)


---
### **d) Como veo que permisos tiene un archivo?**

> Ejecutando ls -l "nombre de archivo" ej: `ls -l hola.sh`
