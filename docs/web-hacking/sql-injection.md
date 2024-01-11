---
layout: default
title: SQL Injection
nav_order: 1
parent: Web Hacking
permalink: /web-hacking/sql-injection
---

# SQL Injection (SQLi)
{: .fs-7 }
{: .no_toc }

<br>

La inyección SQL es una vulnerabilidad web que afecta a las bases de datos de una página, donde un atacante de manera mal intencionada, envía a la base de datos consultas que las interpreta como propias, cuando en realidad son realizadas por un tercero que no tiene acceso o permisos de poder realizar esas acciones.
<br>

Es decir, por lo general un usuario común que visita una página web de compras, por ejemplo, realiza inconscientemente peticiones a la base de datos cuando quiere ver **x** producto, stock del producto o publicar un producto, todo esto viaja a través de una petición web hacia el servidor indicándole a la base de datos que debe realizar tal accion. El problema aquí es que el usuario común no crea o formula esas consultas porque ya está programada para hacer **x** acción por los desarrolladores, sino que un usuario con conocimiento y mala intención, pueda crear **sus propias consultas** con la sintaxis adecuada para esa base de datos. Una vez encontrada la vulnerabilidad, el atacante determina que tipo de inyección es y como se puede explotar para modificar o actualizar datos, eliminarlos por completo, robar información como datos de empleados, credenciales de usuarios, datos de socios, etc.
<br>

Con esto explicado veremos los distintos tipos de inyecciones SQL que hay y como se explotan, cabe recalcar que no explicaré temas como la sintaxis de bases de datos, de como se crean, modifican, eliminan o agregan datos, eso puede investigarlo por su cuenta.
<br>

<details open markdown="block">
  <summary>
    Tipos de SQL Injection
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

# In-band SQL Injection

Las inyecciones SQL en banda son un tipo de inyección en las que el output o salida de la consulta que se le realiza a la base de datos, se aprecia en la misma página, es decir el frontend de la página, ósea la interfaz de usuario.

---

# Blind SQL Injection

Una inyección de SQL ciega es aquella que no puede verse en la página, es decir que es lo contrario a la inyección en banda, ya que en este caso los datos están siendo manipulados, pero no se muestran en la página, sino que deben obtenerse sin poder verlos en la web.

---

# Otros

Además de las inyecciones ciegas y en banda, podemos realizar otras acciones, como eludir paneles de autenticación, obtener ejecución remota de comandos(**RCE**), ver información de la infraestructura del servidor, etc.