# TP4 Sistemas de Computacion

## Modulos de Kernel

Un módulo del kernel de Linux es un segmento de código capaz de cargarse y descargarse dinámicamente dentro del kernel según sea necesario. Los módulos mejoran las capacidades del kernel sin necesidad de reiniciar el sistema. Un ejemplo notable se ve en el módulo del controlador de dispositivo (drivers), que facilita la interacción del kernel con los componentes de hardware vinculados al sistema.

## Desafío #1 

### ¿Qué es checkinstall y para qué sirve?

CheckInstall es un programa de computadora para Sistemas Operativos Unix-like que permite la instalación y desinstalación de software compilado desde el código fuente para ser administrado por un Sistema de gestión de paquetes . Después de la compilación del paquete este puede generar paquetes compartibles para Slackware-, RPM, o Debian. El paquete de generado puede ser eliminado limpiamente por el Sistema de gestión de paquetes.

Los beneficios principales de CheckInstall contra simplemente ejecutar make install es la habilidad de desinstalar el paquete del sistema usando su Sistema de gestión de paquetes, además de poder instalar el paquete resultante en varias computadoras. 

Frecuentemente ocurre que un programa está sólo disponible como código fuente tar.gz (no hay paquete disponible rpm o Debian). En ese caso se debe descargar el paquete fuente, desempaquetarlo y compilarlo manualmente. Sin embargo, muchas veces, no se provee una manera apropiada para desinstalar el programa. La herramienta CheckInstall fue escrita por Felipe Eduardo Sánchez Díaz Durán para resolver este problema. 

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

El kernel Linux tiene un diseño modular. Al arrancar, solamente se carga en memoria un kernel residente de pequeño tamaño. Desde ese momento, cada vez que el usuario solicita la ejecucion de una funcion no disponible en el kernel residente, el modulo del kernel se carga dinamicamente en memoria. Despues de un periodo especifico de inactividad, el modulo se puede quitar de la memoria.

El mecanismo que posibilita la carga dinamica de los modulos es un subproceso del kernel denominado kmod. Los modulos no se cargan salvo si se necesitan. Cuando el kernel solicita el uso de un modulo, este se carga junto con todas las dependencias. 

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
![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/007ce7a2-8cbd-4e62-b48f-fa2c7cfe485a)

![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/d1cf08a2-8a74-4e92-9e8b-3fe3e00cecc3)

2. Quitar modulo:
```sh
sudo rmmod mimodulo
sudo dmesg
lsmod | grep mod

cat /proc/modules  | grep mod
```
![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/f9529267-8af6-4e04-86c3-d5cde1db61c2)

3. Comparar informacion de modulos
```sh
modinfo mimodulo.ko 
modinfo /lib/modules/$(uname -r)/kernel/crypto/des_generic.ko
```
1. mimodulo.ko
   
   ![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/543bdc44-5aa7-45b6-ae4b-e8679df4f42c)
   
3. des_generic.ko
   
   ![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/041c995f-a6dc-4d14-a0c8-eb2d9c6761ea)
   
### ¿Qué diferencias se pueden observar entre los dos modinfo ? 

El modulo de desgeneric.ko tiene asignados varios Alias, sig_id, signer, sig_key, sig_hashalgo y signature, con las cuales no cuenta mimodulo.ko.  

### ¿Qué divers/modulos estan cargados en sus propias pc? comparar las salidas con las computadoras de cada integrante del grupo. Expliquen las diferencias. Carguen un txt con la salida de cada integrante en el repo y pongan un diff en el informe.
Ejecutando el siguiente comando obtenemos los modulos cargados en la computadora de cada uno de los integrantes:
```sh
lsmod > modules_list.txt
```
### ¿cuales no están cargados pero están disponibles? que pasa cuando el driver de un dispositivo no está disponible. 

Cuando un driver de un dispositivo no está disponible en un sistema Linux, ocurre que el dispositivo no funcionará correctamente haciendo que el sistema operativo no pueda comunicarse con el hardware del dispositivo, lo que resulta en la falta de funcionalidad del dispositivo. Tambien el sistema puede no reconocer el dispositivo en absoluto, lo que significa que no aparecerá en la lista de dispositivos del sistema (lspci, lsusb, etc.).

### Correr hwinfo en una pc real con hw real y agregar la url de la información de hw en el reporte. 
Ejecutando los siguientes comandos obtenemos el reporte del hardware en la computadora:

```sh
sudo apt-get install hwinfo
sudo hwinfo --all > hwinfo_report.txt
```
Los reportes se encuentran disponibles en la carpeta hwinfo_integrantes

### ¿Qué diferencia existe entre un módulo y un programa  ? 

Un módulo representa un segmento de código independiente, diseñado para ser integrado o removido de un programa o sistema operativo, especialmente en entornos basados en Linux. Un ejemplo prominente es el módulo del kernel, que permite extender de las capacidades del núcleo del sistema sin la necesidad de reiniciar. Estos módulos, comúnmente escritos en C, operan dentro del espacio del kernel, otorgándoles acceso inmediato tanto al hardware como a los recursos del sistema operativo.

En contraste, un programa consiste en una serie de instrucciones que se procesan secuencialmente para cumplir con un objetivo determinado. Este puede manifestarse como una aplicación integral operando bajo un sistema operativo y puede estar desarrollado en idiomas de programación variados, incluyendo C, Python o Java. Estas aplicaciones funcionan dentro del espacio de usuario, lo cual implica que se ejecutan con privilegios restringidos, sin interacción directa con los componentes del sistema o el hardware.

### ¿Cómo puede ver una lista de las llamadas al sistema que realiza un simple helloworld en c?
Esta información se puede obtener de la siguiente forma:

```sh
cd checkinstall
make
strace ./hello
```
Se utiliza el comando strace para observar las llamadas al sistemas del siguiente programa:

![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/90f2aaaf-f568-46a5-8453-c27b3a247787)

### ¿Que es un segmentation fault? como lo maneja el kernel y como lo hace un programa?

Un segmentation fault es un error que ocurre cuando un programa intenta acceder a una región de  memoria a la que no tiene permiso o que no existe. Este tipo de error generalmente se produce cuando un programa:

1. Intenta leer o escribir en una dirección de memoria fuera del espacio asignado.
2. Accede a una dirección de memoria que ha sido liberada.
3. Usa un puntero no inicializado o inválido.

Los segmentation faults son comunes en programas escritos en lenguajes como C y C++, donde el manejo de la memoria es manual.

**Manejo del kernel**
Cuando un programa intenta acceder a una memoria inválida, el procesador genera una excepción que es capturada por el kernel del sistema operativo y responde emitiendo una señal de violación de segmento __(SIGSEGV)__ al proceso infractor. Esta alerta comunica que se ha producido un intento de acceso a un área restringida de la memoria. Si el proceso en cuestión no está configurado para gestionar esta señal, el kernel procede a finalizarlo para mantener la estabilidad del sistema y evitar comportamientos erráticos. Durante este proceso, puede generarse un archivo de volcado de memoria _(core dump)_, que sirve para analizar posteriormente la causa exacta del error.

El kernel, utilizando sus estructuras de control como las tablas de páginas, supervisa cada solicitud de acceso a la memoria. En caso de que una solicitud contravenga las políticas establecidas en estas tablas, el hardware interviene, creando una interrupción que el kernel interpreta como un segmentation fault, llevando a cabo las acciones correctivas necesarias.

**Manejo de un programa**
Un programa puede gestionar un Segmentation Fault configurando un manejador de señales para la señal __SIGSEGV__. Esto permite que el programa responda de manera controlada al error, en lugar de terminar abruptamente. Al producirse un Segmentation Fault, es posible que se genere un volcado de memoria _(core dump)_, que contiene información detallada sobre el estado del programa en ese momento. Los desarrolladores utilizan este volcado para identificar y corregir la causa del error. Aunque algunos programas implementan manejadores personalizados para señales como __SIGSEGV__, liberando recursos o registrando el error antes de finalizar, es importante proceder con precaución, ya que el comportamiento tras recibir una señal de __SIGSEGV__ es indefinido según los estándares POSIX.

### ¿Se animan a intentar firmar un módulo de kernel ? y documentar el proceso ?  https://askubuntu.com/questions/770205/how-to-sign-kernel-modules-with-sign-file

Para firmar un módulo antes de cargarlo en Linux, puedes seguir estos pasos:
1. Primero, necesitas generar una clave privada y su correspondiente clave pública:

```sh
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=MyModuleSignatureKey/"
```
2. Luego, necesitas registrar la clave pública con el MOK (Machine Owner Key) en el sistema. Para hacerlo, usa el comando mokutil:
```sh
sudo mokutil --import MOK.der
```
Esto pedirá que crees una contraseña. Esta contraseña se usará para completar el proceso de inscripción cuando reinicies el sistema.
3. Reinicia tu máquina. Durante el proceso de arranque, el sistema debería entrar en el modo de inscripción de MOK. Sigue las instrucciones y proporciona la contraseña que configuraste anteriormente para completar la inscripción de la clave pública.
4. Con la clave privada y pública generadas, ahora puedes firmar tu módulo. Usa el script sign-file proporcionado por el paquete linux-headers.
```sh
/usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der mimodulo.ko
```
5. Finalmente, puedes cargar tu módulo firmado con insmod o modprobe:
```sh
sudo insmod mimodulo.ko
```
Finalmente obtendremos el modulo firmado:

![image](https://github.com/Giuli2803/tp4-siscom/assets/66461191/b2bea3eb-59fa-4d35-aa91-c546d2742b51)

### Agregar evidencia de la compilación, carga y descarga de su propio módulo imprimiendo el nombre del equipo en los registros del kernel. 

Se muestra en la siguiente imagen la compilacion y carga y descarga del modulo, modificado para que imprima el nombre del equipo "Las neuronas de bajo consumo" al cargarse el modulo.

![image](https://github.com/marcosraimondi1/tp4-siscom/assets/69517496/31d936de-80ed-4f9f-9b63-8dd9c17480c7)


### ¿Que pasa si mi compañero con secure boot habilitado intenta cargar un módulo firmado por mi? 

Secure Boot es una característica de seguridad proporcionada por UEFI que garantiza que solo el software firmado y confiable se ejecute durante el arranque del sistema. Esto incluye módulos del kernel, asegurando que no han sido alterados ni firmados por entidades no confiables. Esta medida protege contra malware que intenta ejecutarse en las primeras etapas del arranque del sistema.

Si mi compañero tiene Secure Boot habilitado e intenta cargar un módulo firmado por mí, el resultado dependerá de si la firma del módulo es reconocida y confiable para el firmware de Secure Boot. Si el módulo está firmado con una clave reconocida y confiable por el firmware, debería cargarse sin problemas. Sin embargo, si la firma no es reconocida o hay algún problema con la clave, el firmware bloqueará la carga del módulo.

Para que un módulo firmado personalmente sea aceptado por Secure Boot, generalmente se necesita:
1. Generar una clave privada y un certificado para la firma del módulo
2. Utilizar la clave privada para firmar el módulo, lo que producie un archivo de firma digital que acompañará al módulo
3. Registrar el certificado correspondiente a la clave privada en la base de datos de claves de Secure Boot del sistema de mi compañero. Este proceso generalmente requiere acceso al firmware UEFI de su máquina, a menudo a través de una interfaz de usuario de firmware.

Una vez que el certificado está registrado en la base de datos, Secure Boot podrá verificar la firma digital cuando mi compañero intente cargar el módulo. Si la firma es válida y reconocida, el módulo se cargará correctamente; de lo contrario, la carga del módulo fallará.

En algunos casos, puede ser necesario desactivar temporalmente Secure Boot para cargar módulos no firmados o firmados con claves no reconocidas, aunque esto no es recomendable desde una perspectiva de seguridad. Manipular Secure Boot y firmar módulos requiere un conocimiento técnico avanzado y debe hacerse con cuidado para no comprometer la seguridad del sistema. 
