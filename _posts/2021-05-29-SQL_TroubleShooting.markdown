---
title:  "SQL Troubleshooting"
date:   2021-05-29 19:30:00
categories: [SQL]
tags: [SQL, Microsoft]
---
A veces nos tenemos que enfrentar a ciertos problemas de lentitud de aplicaciones desarrolladas, las cuales a nivel de la experiencia de usuario, resultan ser bastante "lentas".

## Introducción ##

Generalmente en arquitectura de dos o tres capas, nos tenemos que enfrentar a ciertos problemas de lentitud que nos reportan los usuarios. En mi experiencia, me he enfrentado a diferentes desarrollos, .NET, Genexus, Java.
Si bien siempre vamos a ir a revisar como esta de recursos dicha infraestructura, (Memoria, Disco, CPU), en la mayoria de los casos el problema termina en el servidor de base de datos. En este caso SQL Server. A continuacion les voy a mostrar dos herramientas que nos van a ayudar a hacer un poco de throubleshooting.

## Manos a la obra ##

**SQL Management Studio:**

1- Abrimos SQL Management Studio.
2- Nos conectamos a la instancia de SQL.
3- Boton derecho sobre la instancia y luego hacer click en **Activity Monitor**
4- Aqui veremos que tenemos dos cosas importantes a ver:

a- **Recent Expensive Queries** son todas las consultas que estan generando un alto costo a nivel de la instancia de SQL, son las que mas demoran, las que mas consumen algun recurso.

Aqui podremos ordenarlas por la columna **Average duration** y luego hacer clic derecho en la consulta y por ultimo **Show execution plan** aqui pordremos ver todo el plan de ejecucion de la consulta y ver en que parte se esta generando el mayor costo.

Si estamos trabajando con versiones de SQL nuevas como ser SQL2019, tambien nos recomendara crear algun indice, en caso de que falte, dandonos la query para crearlo.

Toda esta informacion se puede guardar y podemos compartirla con el equipo de desarrollo, para que tome acciones y pueda optimizar a nivel del desarrollo dicha consula.

**Database Engine Tunning Advisor**

Esta herramienta nos va a hacer un analisis de performance de la base de datos seleccionada, y nos dara un informe, por ejemplo, nos puede dar en que tablas se estan presentando problemas, nos va a dar una query para crear todos los indices faltantes en dichas tablas.

## Conclusion ##

No soy DBA, pero he tenido que trabajar mucho con problemas de lentitud de diferentes aplicaciones, y estas herramientas me han ayudado a encontrar evidencias para darle como informaicon a los diferentes equipos de desarrollo y que puedan optimizar el desarrollo y como la aplicacion arma las consultas.

## Links de referencia ##

[SQL Activity Monitor][SQL Activity Monitor]

[SQL Activity Monitor]: https://docs.microsoft.com/en-us/sql/relational-databases/performance-monitor/open-activity-monitor-sql-server-management-studio?view=sql-server-ver15

[SQL Database Tunning Advisor][SQL Database Tunning Advisor]

[SQL Database Tunning Advisor]: https://docs.microsoft.com/en-us/sql/tools/dta/tutorial-database-engine-tuning-advisor?view=sql-server-ver15