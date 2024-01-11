---
layout: default
title: Path Traversal
nav_order: 1
parent: Web Hacking
permalink: /web-hacking/path-traversal
---

# En desarollo....

La pagina se encuentra en desarollo, vuelva pronto :)

{% comment %}
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

# Test
{% endcomment %}