---
layout: default
title: Medium
nav_order: 2
parent: TryHackMe
grand_parent: CTFs
permalink: /ctfs/tryhackme/medium
---

# Maquinas MEDIA
{: .fs-7 }
{: .no_toc }

<br>

<details open markdown="block">
  <summary>
    Maquinas
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Prioritise - MEDIUM - Writeup TryHackMe

<br>

![](/assets/images/prioritise.png)

<br>

{: .important-title }
> Tecnicas
>
- Blind SQL injection based on boolean values.
- Python scripting for data dump.

<br>

La máquina prioritise tiene un servicio web corriendo por el puerto **80**, que ofrece una especie de servicio web de "prioridades", algo así como una especie de agenda. Si accedemos a la página veremos algo como lo siguiente:

<br>

![](/assets/images/prioritise_pages.png)

<br>

Si jugamos un poco con la página vemos que podemos agregar registros indicando el título y su fecha, y si ordenamos los registros por título, veríamos los registros por orden alfabético y por fecha seria de menor a mayor. Si ordenamos por título veríamos lo siguiente:

<br>

![](/assets/images/prioritise_url.png)

<br>

Como puede ver el parámetro `order` de la URL ordena los registros basándose en el título o fecha. Si a title le agregamos un **desc** ordenara los títulos de manera descendente al igual que lo haría con date, esto quiere decir que la página interpreta consultas SQL y podríamos probar algo como lo siguiente:

<br>

![](/assets/images/prioritise_test.png)

<br>

En lo personal utilicé BurpSuite por comodidad, pero puede hacerlo desde la página si lo desea. Luego de hacer la prueba, note que si a **AND** le pasaba un valor distinto de 1, la página seguía mostrando lo mismo, por lo que, para determinar que las condiciones se cumplan y sean verdaderas, utilice condicionales que de proporcionar un valor correcto, devolverían un ordenamiento por título y de lo contrario generarían un error:

<br>

![](/assets/images/prioritise_error.png)

<br>

Para generar un error a propósito, haga uso de la función **load_extension()** que lo que hace es cargar una biblioteca de extensión SQLite compartida, que al ser 1 un valor erróneo, es incorrecto por lo que devuelve un "500 Internal Server Error" y se valida la condición.

<br>

Gracias a esto, ahora podemos obtener información de la base de datos carácter por carácter. Para empezar, lo que hago es determinar cuantas tablas hay y de cuanto es la longitud de sus nombres haciendo uso de los siguientes payloads:

<br>

![](/assets/images/prioritise_tablas.png)

<br>

![](/assets/images/prioritise_longitud_de_tablas.png)

<br>

Basándonos en esta información, lo que podríamos hacer es crear un exploit que automatice el dumpeo de datos, ya que hacerlo manualmente carácter por carácter sería bastante tedioso, para ello, en mi github podrá encontrar la [herramienta](https://github.com/N1SEC/exploit_sqli.py) que estoy utilizando:

<br>

![](/assets/images/prioritise_flag.png)

<br>

Si desea ver como saque el nombre de las tablas, columnas y datos, puede leer el script de python donde cargue los payloads que utilice para dumpear la flag.