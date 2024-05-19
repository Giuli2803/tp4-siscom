# TP4 Sistemas de Computacion

## Modulos de Kernel

Un módulo del kernel de Linux es un segmento de código capaz de cargarse y descargarse dinámicamente dentro del kernel según sea necesario. Los módulos mejoran las capacidades del kernel sin necesidad de reiniciar el sistema. Un ejemplo notable se ve en el módulo del controlador de dispositivo (drivers), que facilita la interacción del kernel con los componentes de hardware vinculados al sistema.

## Desafío #1 

### ¿Qué es checkinstall y para qué sirve?

CheckInstall es un programa de computadora para Sistemas Operativos Unix-like que permite la instalación y desinstalación de software compilado desde el código fuente para ser administrado por un Sistema de gestión de paquetes . Después de la compilación del paquete este puede generar paquetes compartibles para Slackware-, RPM, o Debian. El paquete de generado puede ser eliminado limpiamente por el Sistema de gestión de paquetes.

Los beneficios principales de CheckInstall contra simplemente ejecutar make install es la habilidad de desinstalar el paquete del sistema usando su Sistema de gestión de paquetes, además de poder instalar el paquete resultante en varias computadoras. 

Frecuentemente ocurre que un programa está sólo disponible como código fuente tar.gz (no hay paquete disponible rpm o Debian). En ese caso se debe descargar el paquete fuente, demepaquetarlo y compilarlo manualmente. Sin embargo, muchas veces, no se provee una manera apropiada para desinstalar el programa. La herramienta CheckInstall fue escrita por Felipe Eduardo Sánchez Díaz Durán para resolver este problema. 

En un programa compatible con GNU Autoconf se instala el programa (luego de descargado y desempaquetado el paquete fuente) con la secuencia:
```sh
./configure && make && make install.
```

En cambio con checkinstall:
```sh
./configure && make && checkinstall
```

### ¿Se animan a usarlo para empaquetar un hello world ? 

Para hacer esto se debe crear un archivo .c con su respectivo makefile. Posteriormente se hace make y se utiliza checkinstall.Posteriormente se puede utilzar el gestor de paquetes del sistema para instalar el programa. 

### Revisar la bibliografía para impulsar acciones que permitan mejorar la seguridad del kernel, concretamente: evitando cargar módulos que no estén firmados. ¿Qué otras medidas de seguridad podemos implementar para asegurar nuestro ordenador desde el kernel ? 

enlace:  https://access.redhat.com/documentation/es-es/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/signing-kernel-modules-for-secure-boot_managing-kernel-modules

Si el Arranque Seguro está habilitado, los cargadores de arranque del sistema operativo UEFI, el kernel de Red Hat Enterprise Linux y todos los módulos del kernel tienen que ser firmados con una clave privada y autenticados con la clave pública correspondiente. Si no están firmados y autenticados, el sistema no podrá terminar el proceso de arranque. 

Requisitos:
Se necesita generar un par de claves X.509 públicas y privadas para tener éxito en utilizar módulos del kernel en un sistema habilitado para el Arranque Seguro. Se utilizará la clave privada para firmar el módulo del kernel. También tendrá que añadir la clave pública correspondiente a la Clave del Propietario de la Máquina (MOK) para el Arranque Seguro para validar el módulo firmado. 

Otras medidas de seguridad que se pueden implementar:
- Habilitar SELinux o AppArmor: Sistemas de control de acceso obligatorio que proporcionan políticas de seguridad estrictas.
- Usar Grsecurity/PaX: Conjunto de parches de seguridad que mejoran significativamente la seguridad del kernel.
- Habilitar Secure Boot: Asegura que el sistema arranca con software de confianza.
- Configurar kernel con KSPP (Kernel Self Protection Project): Implementa protecciones avanzadas contra vulnerabilidades.
- Deshabilitar SysRq: Reduce el riesgo desactivando la combinación de teclas que interactúa directamente con el kernel.
- Deshabilitar el acceso a dispositivos USB: Minimiza el vector de ataque restringiendo el uso de dispositivos USB.
- Implementar auditoría de seguridad: Utiliza un sistema de auditoría para monitorizar y registrar eventos de seguridad del sistema.

## Desafío #2 Modulos vs Programas

enlace: https://sysprog21.github.io/lkmpg/#preliminaries

### ¿Cómo empiezan y terminan unos y otros ?

Un **programa** típico comienza con una función main(), ejecuta una serie de instrucciones y termina después de completar estas instrucciones. Los módulos del kernel, sin embargo, siguen un patrón diferente. Un **módulo** siempre comienza con la función init_module o con una función designada por la llamada module_init. Esta función actúa como el punto de entrada del módulo, informando al kernel de las funcionalidades del módulo y preparando al kernel para utilizar las funciones del módulo cuando sea necesario. Después de realizar estas tareas, la función de entrada retorna, y el módulo permanece inactivo hasta que el kernel requiera su código.

Todos los módulos concluyen invocando cleanup_module o una función especificada a través de la llamada module_exit. Esto sirve como la función de salida del módulo, revirtiendo las acciones de la función de entrada al anular el registro de las funcionalidades previamente registradas.

Es obligatorio que cada módulo tenga tanto una función de entrada como una de salida.

### ¿Qué funciones tiene disponible un programa y un módulo ?

Un programa puede usar funciones definidas en librerias (como por ejemplo printf de la libreria estandar de C) que luego en el momento del linkeo se agrega al programa.

Por otro lado, los módulos son archivos de objeto cuyos símbolos se resuelven al ejecutar insmod o modprobe. La definición de los símbolos proviene del propio kernel; las únicas funciones externas que se pueden usar son las proporcionadas por el kernel. 

Se pueden observar los símbolos que han sido exportados por el kernel en /proc/kallsyms:
![image](https://github.com/marcosraimondi1/tp4-siscom/assets/69517496/57de5f3a-f320-453d-848d-b91f1da496f3)

<small>Se utiliza sudo para poder observar la direccion de memoria de los simbolos, ya que para un usuario no root esa direccion no es visible.</small>

El formato de las entradas es el siguiente: <dirección> <tipo> <nombre_del_símbolo>

- Dirección: La dirección de memoria en la que se encuentra el símbolo.
- Tipo: Un carácter que indica el tipo de símbolo (por ejemplo, T para una función de texto (código), D para datos, A indica que tiene una direccion absoluta).
- Nombre del símbolo: El nombre del símbolo exportado

La diferencia entre las funciones de biblioteca y las llamadas al sistema es que las funciones de biblioteca son de nivel superior, se ejecutan completamente en el espacio de usuario y proporcionan una interfaz más conveniente para el programador a las funciones que realizan el trabajo real: las llamadas al sistema. Las llamadas al sistema se ejecutan en modo kernel en nombre del usuario y son proporcionadas por el propio kernel. 

La función de biblioteca printf() puede parecer una función de impresión muy general, pero todo lo que realmente hace es formatear los datos en cadenas y escribir los datos de la cadena usando la llamada al sistema de bajo nivel write(), que luego envía los datos a la salida estándar.

Se utiliza el comando strace para observar las llamadas al sistemas del siguiente programa
```C
#include <stdio.h>
int main(int argc, char *argv[]) {
printf("hello world\n");
return 0;
}
```
Salida:
![image](https://github.com/marcosraimondi1/tp4-siscom/assets/69517496/d3d46d7b-e5c7-4836-9153-96f23e7d84ba)

### ¿Quien carga los modulos del kernel?

Los módulos del kernel son cargados por el programa insmod o modprobe. Estos son utilitarios de línea de comandos que permiten cargar módulos del kernel en el espacio de kernel en ejecución:
- **insmod**: Es un comando básico que carga módulos del kernel directamente. Requiere que especifiques la ruta completa del módulo que se desea cargar, así como cualquier argumento adicional que el módulo pueda necesitar.
- **modprobe**: Es una herramienta más avanzada y versátil que insmod. Además de cargar módulos, modprobe maneja automáticamente las dependencias del módulo, cargando cualquier otro módulo necesario. También proporciona una configuración más flexible y opciones adicionales para la administración de módulos.

Para ejecutar estas funciones se deben llamar con sudo ya que requieren permisos de usuario root.

### Espacio de usuario vs espacio del kernel.

El kernel principalmente gestiona el acceso a recursos. Los programas compiten frecuentemente por los mismos recursos. El papel del kernel es mantener el orden, asegurando que los usuarios no accedan a los recursos indiscriminadamente.

Para gestionar esto, las CPU operan en diferentes modos, cada uno ofreciendo diferentes niveles de control del sistema. La arquitectura Intel 80386, por ejemplo, contaba con cuatro de estos modos, conocidos como anillos. Sin embargo, Unix utiliza solo dos de estos anillos: el anillo más alto (anillo 0, también conocido como "modo supervisor", donde todas las acciones son permitidas) y el anillo más bajo, conocido como "modo usuario".

![image](https://github.com/marcosraimondi1/tp4-siscom/assets/69517496/4936d16b-d3ba-45a2-8e43-58ec0691bd6d)

Las funciones de una libreria se usan en en modo usuario. Esa funcion luego llama a una o más llamadas al sistema, y estas llamadas al sistema se ejecutan en nombre de la función, pero lo hacen en modo supervisor ya que forman parte del propio kernel. Una vez que la llamada al sistema completa su tarea, devuelve el control y la ejecución vuelve al modo usuario.

### Espacio de datos y espacio de codigo para usuario y kernel. Riesgos!!

**Espacio de Datos o namespace**: 

Al escribir un programa en C, se utilizan variables que son convenientes y tienen sentido para el lector. Sin embargo, en grandes proyectos, cualquier variable global que será parte de una comunidad de variables globales de otras personas; y algunos nombres de variables pueden entrar en conflicto. En proyectos grandes, se debe hacer un esfuerzo para recordar los nombres reservados y encontrar formas de desarrollar un esquema para nombrar variables y símbolos únicos.

Al escribir código del kernel, incluso el módulo más pequeño se vinculará con todo el kernel, por lo que esto definitivamente es un problema. La mejor manera de abordarlo es declarar todas tus variables como estáticas y utilizar un prefijo bien definido para los símbolos. Por convención, todos los prefijos del kernel están en minúsculas. Otra opción es declarar una tabla de símbolos y registrarla en el kernel.

El archivo /proc/kallsyms contiene todos los símbolos que el kernel conoce y que, por lo tanto, son accesibles para tus módulos ya que comparten el espacio de código del kernel.

**Espacio de Codigo o codespace**:

Cuando se crea un proceso, el kernel reserva una porción de memoria física real y se la entrega al proceso para que la utilice en su código, variables, pila y heap. Esta memoria comienza con 0x00000000 y se extiende hasta donde sea necesario. Dado que el espacio de memoria para dos procesos no se superpone, cada proceso que puede acceder a una dirección de memoria, digamos 0xbffff978, estaría accediendo a una ubicación diferente en la memoria física real ya que esa direccion representa un desplazamiento en el bloque de memoria que se le fue asignado y resulta en una direccion de memoria fisica diferente para cada proceso.

El kernel también tiene su propio espacio de memoria. Dado que un módulo es código que puede ser insertado y eliminado dinámicamente en el kernel, comparte el espacio de código del kernel en lugar de tener el suyo propio. Por lo tanto, si el módulo provoca un segmentation fault, el kernel también lo hará. Y si sobrescribe datos debido a un error (accidental o intencionadamente), entonces se estara pisando datos (o código) del kernel.

### Drivers. Investigar contenido de /dev.

Una clase de módulo es el device driver (controlador de dispositivo), proporciona funcionalidad para hardware como un puerto serial. En Unix, cada pieza de hardware está representada por un archivo ubicado en **/dev** llamado archivo de dispositivo, que proporciona los medios para comunicarse con el hardware. El driver proporciona la comunicación en nombre de un programa de usuario. 

De esta forma un programa de espacio de usuario puede user un dispositivo de hardware a traves de estos drivers sin preocuparse por el hardware especifico que esta utilizando.

Los dispositivos se dividen en dos tipos: dispositivos de caracteres y dispositivos de bloques. La diferencia es que los dispositivos de bloque tienen un búfer para solicitudes, por lo que pueden elegir el mejor orden en el que responder a las solicitudes. Esto es importante en el caso de dispositivos de almacenamiento, donde es más rápido leer o escribir sectores que están cerca uno del otro, en lugar de los que están más separados. Otra diferencia es que los dispositivos de bloque solo pueden aceptar entrada y devolver salida en bloques (cuyo tamaño puede variar según el dispositivo), mientras que los dispositivos de caracteres pueden usar tantos o tan pocos bytes como deseen. La mayoría de los dispositivos son de caracteres, porque no necesitan este tipo de almacenamiento en búfer y no operan con un tamaño de bloque fijo.

Para saber si un archivo de dispositivo es para un dispositivo de bloque o de caracteres, se puede ver el primer carácter en la salida de ls -l. Si es 'b', entonces es un dispositivo de bloque, y si es 'c', entonces es un dispositivo de caracteres.

Ejemplo de salida de ls -l /dev:

![image](https://github.com/marcosraimondi1/tp4-siscom/assets/69517496/fa90e6b8-e231-4721-a088-5dac381bac02)

Se puede observar que el dispositivo /dev/sda y sus particiones son dispositivos de bloque. Corresponde al disco de estado solido de la PC y sus respectivas particiones.

Los archivos /dev/tty en Linux son dispositivos de caracteres que representan las terminales del sistema y proporcionan un medio para interactuar con ellas desde programas en el sistema.

Las dos columnas de numeros separadas por coma corresponde al número mayor y numero menor del dispositivo. El número mayor indica qué controlador se usa para acceder al hardware. A cada controlador se le asigna un número mayor único; todos los archivos de dispositivo con el mismo número mayor son controlados por el mismo controlador. El número menor es utilizado por el controlador para distinguir entre el hardware variado que controla. 

Se observa en el ejemplo que en la entrada de /dev/sda son controlados por el mismo driver ya que tienen el mismo numero mayor 8. Ese driver distingue lo que son las particiones por el numero menor de cada una.

## Desafío #3

Seguir los siguientes pasos:

1. Compilar y cargar el modulo "mimodulo.ko":
```sh
cd part1
make
sudo insmod mimodulo.ko
sudo dmesg
lsmod | grep mod
```

2. Quitar modulo:
```sh
sudo rmmod mimodulo
sudo dmsg
lsmod | grep mod

cat /proc/modules  | grep mod
```

3. Comparar informacion de modulos
```sh
modinfo mimodulo.ko 
modinfo /lib/modules/$(uname -r)/kernel/crypto/des_generic.ko
```

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

