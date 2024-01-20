---
layout: default
title: LFI-RFI
nav_order: 2
parent: Web Hacking
permalink: /web-hacking/LFI-RFI
---

# Local File Inclusion(LFI) / Remote File Inclusion(RFI)
{: .fs-7 }
{: .no_toc }
<br>

Las vulnerabilidades de inclusión de archivos permite a un atacante poder ver archivos del sistema y datos, que se encuentren expuestos y con permisos de lectura en el servidor.

<br>

La inclusión de archivos puede hacerse de 2 maneras, una por medio del LFI y otra por RFI, la diferencia radica en que el **Local File Inclusion** se hace, como lo dice la palabra, de forma local es decir que podemos ver archivos locales de la máquina manipulando la página y sus peticiones. 

<br>

Al contrario, en RFI, nosotros debemos pasarle un código malicioso **remotamente** para que el servidor lo interprete y ejecute el payload. Este payload malicioso se configura de manera local y se carga desde un servidor web controlado por el atacante, donde la web víctima, interpreta el código malicioso que se está compartiendo desde el servidor atacante.

---

# LFI / RFI exploitation
{: .fs-7 }
{: .no_toc }
<br>

Para llevar a cabo estas explotaciones, debe saber que hay dos funciones que deben estar habilitadas en el servidor, que son `url_allow_fopen` y `url_allow_include`, no necesariamente sabrá esto a no sé que se encuentre expuesto el `phpinfo.php` de la página, pero si no lo esta puede igualmente intentar probar la inclusión de archivos, porque nunca sabré si están habilitadas o no estas opciones. 

<br>

Dicho esto veamos algunas de las explotaciones más típicas que suelen hacerse cuando hay vulnerabilidades de este tipo y como se llevan a cabo.

<details open markdown="block">
  <summary>
    Vulnerabilidades mas tipicas
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

### LFI Enumeration
<br>

Lo primero que haría si encontrara una vulnerabilidad de Local File Iclusion, seria enumerar rutas conocidas manualmente como:


+ /etc/passwd
+ /etc/hosts
+ /proc/self/fd
+ /proc/sched_debug
+ /proc/self/environ
+ /etc/shadow
+ /etc/issue
+ /etc/group
+ /etc/hostname

Puede hacer esto o buscar automáticamente con wfuzz o cualquier otra herramienta de fuzzing, que le permita enumerar archivos del sistema, como por ejemplo:

`wfuzz -c -t 100 -w /usr/share/wordlists/wordlist.txt http://website.com/index.php?file=FUZZ`

---

### LFI to RCE by Log Poisoning
<br>

Podríamos envenenar él registró de logs de servidores **apache** o **nginx**, para obtener ejecución remota de comandos de la siguiente manera:

<br>

![](/assets/images/normal.png)

![](/assets/images/modificado.png)

<br>

Modificaríamos el `User-Agent` de la página web y luego por medio de un LFI, nos aprovecharíamos de esa **lectura** de archivos, para ver los logs y los comandos inyectados:

<br>

![](/assets/images/RCE.png)

---

### LFI to RCE by SSH log poisoning
<br>

Si apuntáramos a la ruta de `/var/log/auth.log`, en caso de estar disponible y en esa ubicación, podríamos ver logs de SSH:

<br>

![](/assets/images/logs.png)

<br>

Podríamos hacer una conexión por `Telnet` o `SSH` a la ip víctima por el puerto que esté corriendo el SSH y cargar un payload malicioso que nos permita ejecución remota de comandos:

<br>

![](/assets/images/shell.png)

<br>

Posterior a eso, si por ejemplo le pasamos el comando `id` al auth.log por medio del parámetro 0, veríamos algo como lo siguiente:

![](/assets/images/command.png)

---

### LFI to RCE by SMTP
<br>

También podemos abusar de protocolos como el **SMTP**, que se utiliza para la manipulación y envío de correos electrónicos. Para explotarlo, tendríamos que ver primero si está corriendo el servicio en el servidor, esto es fácil de saber si realizamos un escaneo de puertos:

<br>

![](/assets/images/port-open.png)

<br>

Luego enviaríamos un correo falso a un usuario registrado en el equipo, y cargaríamos el payload malicioso en la DATA del correo:

<br>

![](/assets/images/e-telnet.png)

<br>

Ahora si por medio del LFI podemos cargar los registros de email del usuario y pasamos un comando del sistema, podremos ver lo siguiente:

<br>

![](/assets/images/id.png)

---

### LFI to RCE by /proc/self/fd/*

El `/proc/self/fd`, es un directorio que almacena los descriptores de archivos E/S(entrada/salida), de programas que interactúan con archivos del sistema y los registran por medio de enlaces simbólicos en el directorio "FD". Lo que sucede es que si fuzzeamos dentro de este directorio por medio de un LFI, podremos dar con logs de apache o nginx, y podría suceder algo como lo siguiente:

<br>

![](/assets/images/fd.png)

<br>

![](/assets/images/burp.png)

<br>

![](/assets/images/9.png)

<br>

Lo que está pasando es que cuando fuzzeamos con **BurpSuite** en el directorio FD, podemos ver varios descriptores de archivos y entre ellos el proceso que manipula apache2, en este caso el archivo access.log, que si lo cargamos veríamos lo siguiente:

<br>

![](/assets/images/logs_proc.png)

<br>

Lo que quedaría ahora, seria hace lo que ya vimos en el apartado **LFI to RCE by Log Poisoning**, cargamos un payload al User-Agent y después por medio del LFI, inyectamos comandos del sistema:


<br>

![](/assets/images/0.png)

<br>

![](/assets/images/RCE_proc.png)

---

### RFI to RCE
<br>

Para explotar un remote file inclusion, debemos contar con un servidor que este bajo nuestro control, porque será por medio del que cargaremos nuestros códigos maliciosos para que la web vulnerable, pueda interpretarlos, por ejemplo:

![](/assets/images/payload_RFI.png)

![](/assets/images/peti.png)

![](/assets/images/shell_RFI.png)

<br>

Como puede apreciar, corremos un servidor en python3 para exportar nuestro archivo malicioso, que la página web interpretara y cargara, lo que nos permitirá inyectar comandos del sistema:

<br>

![](/assets/images/RCE_RFI_1.png)

---

### Use of wrappers
<br>

En ocasiones tendremos casos donde el contenido que queramos ver por LFI, no se interpretara en la página, por lo que podremos hacer uso de envoltorios, que nos permitiría obtener una cadena codificada en base64 que podremos decodificar, para ver el contenido que antes no podíamos, como por ejemplo el contenido de una página en php:

<br>

![](/assets/images/wrapper.png)

![](/assets/images/base64.png)

<br>

Esto, por ejemplo, nos permitiría ver códigos fuentes del servidor y como se está manejando la página por detrás. No solo podemos utilizar el envoltorio php, sino que tenemos otros como:

+ zip://
+ rar://
+ data://
+ expect://
+ input://
+ phar://

Como puede ver esto es un **poco** de lo que puede explotar por medio de la inclusión de archivos, y puede investigar más aun el tema, debido a que hay varias formas de obtener RCE por medio de LFI y puede aprovechas los wrappers antes mencionados para realizar distintas explotaciones.