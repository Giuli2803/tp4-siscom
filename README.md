# TP4 Sistemas de Computacion

## Modulos de Kernel

## Desafío #1 

### ¿Qué es checkinstall y para qué sirve?
### ¿Se animan a usarlo para empaquetar un hello world ? 
### Revisar la bibliografía para impulsar acciones que permitan mejorar la seguridad del kernel, concretamente: evitando cargar módulos que no estén firmados.

enlace:  https://access.redhat.com/documentation/es-es/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/signing-kernel-modules-for-secure-boot_managing-kernel-modules

## Desafío #2

enlace: https://sysprog21.github.io/lkmpg/#preliminaries

### ¿Cómo empiezan y terminan unos y otros ?
### ¿Qué funciones tiene disponible un programa y un módulo ?

Programa: Primero construir un programa que llame a printf(), compilarlo (con opción -Wall y ejecutar strace (strace -tt y strace -c). Explicar lo que se observa.!!

Modulo: La definición de los símbolos proviene del propio kernel; las únicas funciones externas que puede utilizar un módulo, son las proporcionadas por el kernel. Investigar el contenido del archivo /proc/kallsyms .!!

### ¿Quien carga los modulos del kernel?
### Espacio de usuario vs espacio del kernel.
### Espacio de datos y espacio de codigo para usuario y kernel. Riesgos!!
### Drivers. Investigar contenido de /dev.

## Desafío #3

### ¿Qué diferencias se pueden observar entre los dos modinfo ? 
### ¿Qué divers/modulos estan cargados en sus propias pc? comparar las salidas con las computadoras de cada integrante del grupo. Expliquen las diferencias. Carguen un txt con la salida de cada integrante en el repo y pongan un diff en el informe.
### ¿cuales no están cargados pero están disponibles? que pasa cuando el driver de un dispositivo no está disponible. 
### Correr hwinfo en una pc real con hw real y agregar la url de la información de hw en el reporte. 
### ¿Qué diferencia existe entre un módulo y un programa  ? 
### ¿Cómo puede ver una lista de las llamadas al sistema que realiza un simple helloworld en c?
### ¿Que es un segmentation fault? como lo maneja el kernel y como lo hace un programa?
### ¿Se animan a intentar firmar un módulo de kernel ? y documentar el proceso ?  https://askubuntu.com/questions/770205/how-to-sign-kernel-modules-with-sign-file
### Agregar evidencia de la compilación, carga y descarga de su propio módulo imprimiendo el nombre del equipo en los registros del kernel. 
### ¿Que pasa si mi compañero con secure boot habilitado intenta cargar un módulo firmado por mi? 

