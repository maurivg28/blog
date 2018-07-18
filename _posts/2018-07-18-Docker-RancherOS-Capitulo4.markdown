---
title:  "Rancher OS - Capitulo 4 - Monitoreo de contenedores"
date:   2018-07-18 13:26
categories: [Docker, RancherOS, Swarm]
tags: [Docker, DevOps, Contenedores, Images, RancherOS, Swarm]
---
Continuando con el capítulo 4 de Rancher, les quiero contar acerca del monitoreo de contenedores. A continuacion vamos a hablar sobre un amplio abanico de plugins que cuenta Rancher OS, el cual nos va a permitir configurar diferentes herramientas de monitoreo sobre una infraestructura de contenedores que tengamos montada.

![RancherOS_Logo]({{ site.baseurl }}/images/rancheros_logo1.png)

## Comparacion de soluciones de monitoreo para Rancher ##

Los entornos de monitoreo que podremos encontrar son muy variados. Algunos son de código abierto, otros vienen en el código de Rancher y otros son soluciones de terceros que podemos acoplar a la infraestructura montada sobre Rancher. Algunas de estas soluciones de monitoreo estan alojadas en la nube, mientras que otras requieren que sean instaladas en los propios host del clúster. En este post vamos a repasar 10 soluciones de monitoreo pensadas para el monitoreo de contenedores.
Cabe aclarar que las soluciones de monitoreo que vamos a repasar evolucionan y cambian constantemente, por lo cual vamos a dar una mirada muy general de cada una de ellas.


### Soluciones de monitoreo ###

Las solcuiones de monitoreo que vamos a repasar son las siguientes: 

- Native Docker
- cAdvisor
- Scout
- Pingdom
- Datadog
- Sysdig
- Prometheus
- Heapster / Grafana
- ELK stack
- Sensu

Tal vez en proximos capitulos me detenga en contarles como funciona particularmente alguna de estas.


### Comparación general ###

El desafío al comparar objetivamente las soluciones de monitoreo es que las arquitecturas, capacidades, modelos de implementación y costos pueden variar ampliamente. Por ejemplo, mientras que una solución puede extraer y graficar métricas relacionadas con Docker desde un solo host, otra agrega datos de muchos hosts, mide los tiempos de respuesta de las aplicaciones y envía alertas automáticas bajo condiciones particulares. La idea de este artículo es tener un marco útil basado en algunos marcos de funcionalidad, para asi poder comparar cada una de ellas.

![RancherOS_Monitoring_Solutions]({{ site.baseurl }}/images/seven_layers_of_docker_monitoring.png)

- **Hosts Agents:** El agente de host representa los "brazos y las piernas" de la solución de monitoreo, extrayendo datos de series de tiempo de varias fuentes como API y archivos de registro. Los agentes generalmente se instalan en cada host del clúster (ya sea en las instalaciones o en la nube) y, a menudo, se empaquetan como contenedores Docker para facilitar su implementación y administración.

- **Data gathering framework:** Si bien las métricas de un solo host a veces son útiles, los administradores probablemente necesiten una vista consolidada de todos los hosts y aplicaciones. Las soluciones de supervisión suelen tener algún mecanismo para recopilar datos de cada host y persistir en un almacén de datos compartido.

- **Datastore:** El datastore puede ser una base de datos tradicional, pero más comúnmente es una forma de base de datos distribuida y escalable optimizada para datos de series temporales compuestas de pares clave-valor. Algunas soluciones tienen almacenes de datos nativos, mientras que otras aprovechan las fuentes de datos conectables de código abierto.

- **Aggregation engine:** El problema con el almacenamiento de métricas en bruto de docenas de hosts es que la cantidad de datos puede ser abrumadora. Los marcos de monitoreo a menudo brindan capacidades de agregación de datos, procesando datos brutos periódicamente en métricas consolidadas (como resúmenes cada hora o diarios), depurando datos viejos que ya no son necesarios o redefiniendo los datos de alguna manera para respaldar consultas y análisis anticipados.

- **Filtering & Analysis:** Una solución de monitoreo es tan buena como las estadísticas que puede obtener de los datos. Las capacidades de filtrado y análisis varían ampliamente. Algunas soluciones admiten algunas consultas preempaquetadas presentadas como simples gráficos de series de tiempo, mientras que otras tienen paneles personalizables, lenguajes de consulta integrados y sofisticadas funciones analíticas.

- **Visualization tier:** Las herramientas de monitoreo generalmente tienen un nivel de visualización donde los usuarios pueden interactuar con una interfaz web para generar gráficos, formular consultas y, en algunos casos, definir condiciones de alerta. El nivel de visualización puede estar estrechamente relacionado con la funcionalidad de filtrado y análisis, o puede estar separado dependiendo de la solución.

- **Alerting & Notification:** Pocos administradores tienen tiempo para sentarse y monitorear gráficos todo el día. Otra característica común de los sistemas de monitoreo es un subsistema de alerta que puede proporcionar una notificación si se alcanzan o superan los umbrales predefinidos.

Más allá de comprender cómo cada solución de monitoreo implementa las capacidades básicas mencionadas anteriormente, los usuarios también estarán interesados ​​en otros aspectos de la solución de monitoreo:

- **Completeness of the solution**
- **Ease of installation and configuration**
- **Details about the web UI**
- **Ability to forward alerts to external services**
- **Level of community support and engagement (for open-source projects)**
- **Availability in Rancher Catalog**
- **Support for monitoring non-container environments and apps**
- **Native Kubernetes support (Pods, Services, Namespaces, etc.)**
- **Extensibility (APIs, other interfaces)**
- **Deployment model (self-hosted, cloud)**
- **Cost, if applicable**

### Comparación de las 10 soluciones de monitoreo ###

El siguiente diagrama se muestra una vista de alto nivel de cómo nuestras 10 soluciones de monitoreo se relacionan con nuestro modelo de siete capas, qué componentes implementan las capacidades en cada capa y dónde residen los componentes. Cada marco es complicado de explicar, y esta es una simplificación o resumen, pero sin duda, proporciona una vista útil de qué componente hace qué.

![RancherOS_Monitoring_Solutions_comp_fig1]({{ site.baseurl }}/images/gord_comparing_ten_monitoring_solutions-1024x585.png)

A continuación se presentan los atributos adicionales de cada solución de monitoreo. Para algunas soluciones, existen múltiples opciones de implementación, por lo que las comparaciones se vuelven un poco más matizadas.

![RancherOS_Monitoring_Solutions_comp_fig2]({{ site.baseurl }}/images/gord_monitoring_functionality_comparison-1024x640.png)

En un próximos capítulos les voy a contar al detalle sobre la implementacion de alguna de estas 10 solcuiones de monitoreo de contenedores. 

En eñ siguiente capítulo les voy a hablar sobre la implementación de DataDog.

Espero que les sea útil este artículo.

### Links de interés: ###

[Docker Official Page][Docker]

[Rancher OS Official Page][RancherOS]

[Docker]: https://www.docker.com/

[RancherOS]: https://rancher.com/rancher-os/


