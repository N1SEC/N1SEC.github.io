---
layout: default
title: Path Traversal
nav_order: 1
parent: Web Hacking
permalink: /web-hacking/path-traversal
---

# Path Traversal
{: .fs-7 }
{: .no_toc }

<br>

El **path traversal** o **directory traversal**, es una vulnerabilidad que nos permite retroceder o salir del directorio actual en el que nos encontramos, para poder enumerar o ver información del servidor fuera del contexto donde está ubicada la aplicación web. La información que podemos ver puede ser:

* Códigos fuente y datos de la aplicación.
* Credenciales para sistemas back-end.
* Archivos sensibles del sistema operativo.

Esto sucede porque el servidor tiene configurado de mala manera en su código backend algún parámetro, de la página, que nos permite poder salir del directorio que almacena la página web, provocando que podamos "volver" o "subir" un directorio más atrás hasta la raíz, para después poder acceder y ver archivos a los que tengamos permisos de lectura.
<br>

Cabe recalcar que esta vulnerabilidad, va muy de la mano de las **Local File Inclusion** o **LFI**, que son las que nos permiten realizar **esa** lectura de archivos. Por si solo el directory traversal nos permite esa "movilidad" entra la estructura de directorios, pero el LFI es quien nos permite la lectura de los archivos, es por eso que ambos van muy de la mano.

---

# Bypassing filters and preventions
{: .fs-7 }
{: .no_toc }
<br>

Una de las formas en que se evitan este tipo de vulnerabilidades, es aplicando filtros en los parámetros de la URL, que evitan que podamos realizar estas acciones, pero aquí veremos como podemos evitar o saltar estos filtros y prevenciones.

<details open markdown="block">
  <summary>
    Evitar filtros y prevenciones de seguridad
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

### Blocked transverse sequences
<br>

En este caso simple es posible ver archivos del sistema indicando la ruta absoluta, debido a que el aplicativo web, está aplicando un filtro que bloquea **secuencias transversales** lo que deriva en que podamos interpretar como absoluta las rutas de archivos en el servidor, por ejemplo:

<br>

![](/assets/images/blocked_transverse_sequences_1.png)

<br>

Como se puede apreciar en la imagen el parámetro `pl`, recibe la ruta absoluta **/etc/passwd** que es la encargada de mostrar los usuarios registrados en el servidor, la salida de esta petición se vería de la siguiente manera:

<br>

![](/assets/images/blocked_transverse_sequences_2.png)

---

### Simple transverse sequence
<br>

En la siguiente situación podemos intuir que la aplicación web está aplicando un filtro que bloquea la lectura de archivos arbitrarios de manera absoluta, es aquí donde podemos aplicar **secuencias transversales**, de la siguiente manera:

<br>

![](/assets/images/simple_case_of_transverse_sequences.png)

<br>

Como puede apreciar el parámetro `filename` intenta cargar archivos de imagen, pero si le indicamos una secuencia transversal llamando al **passwd**, nos muestra lo siguiente:

<br> 

![](/assets/images/error.png)

<br>

El parámetro `filename` está intentando cargar un recurso de tipo imagen, pero nosotros estamos indicando un archivo propio del sistema linux, para poder ver el contenido de **passwd** podemos interceptar la solicitud con **BurpSuite** y podremos ver el contenido de la siguiente manera:

<br>

![](/assets/images/simple_case_of_transverse_sequences_2.png)

---

### Nested transverse sequences
<br>

Las **secuencias transversales anidadas**, son combinaciones de secuencias transversales juntas y pueden verse de la siguiente manera:

+  `.././`
+  `....////`
+  `..../`
+  `....\/`

<br>

En el siguiente ejemplo podemos ver una secuencia transversal anidada:

![](/assets/images/nested_transverse_sequences.png)

---

### URL-encoded transverse sequences
<br>

Otra forma de evadir filtros de path traversal es codificando las secuencias transversales en URL-encode, evitando filtros de desinfección, por ejemplo:

<br>

![](/assets/images/url-e.png)

<br>

En este caso se codifica el carácter `/`, pero podríamos codificar aún más la cadena quedando algo como:

+ `%2e%2e%2f%2e%2e%2fetc%2fpasswd`

o

+ `%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64`

esto evitaría que se apliquen filtros que eviten secuencias transversales como las ya mencionadas.

---

### Transverse sequences with absolute location
<br>

En ocasiones la aplicación puede que llame a un recurso del equipo indicando la ruta absoluta del archivo, más una secuencia transversal, por ejemplo:

<br>

![](/assets/images/path.png)

<br>

Esto sucede porque la aplicación maneja los archivos utilizando las rutas base, por lo que si indicamos estas rutas más una secuencia transversal, podremos listar archivos del servidor.

---

### Transverse sequences with null bytes
<br>

En ocasiones puede haber filtros donde el servidor requiera la extensión de un archivo solicitado y una forma de poder evitar estas extensiones de archivos, es agregando un **NULL Byte** o byte nulo(`%00`), que quita la extensión requerida por el servidor e interpreta él archivó que nosotros solicitemos, por ejemplo:

<br>

![](/assets/images/null.png)

<br>