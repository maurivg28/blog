---
title:  "Azure Front Door"
date:   2018-12-03 23:00:00
categories: [Packer]
tags: [Azure, CDN, LB, Firewall, WAF]
---
En ocubre de 2018 Microsoft anuncio muchas novedades sobre nuevos productos que saldran y que estaran disponibles entre dicho mes y finales de año. Uno de ellos es Azure Front Door, un nuevo producto que combina caracteristicas de productos anteriores como ser, traffic manager, CDN, application gateway. Yo diria que es un "all in one".

![AFD_Logo]({{ site.baseurl }}/images/frontdoor.png)


## Introducción ##

Azure Front Door Service: equilibrio de carga HTTP global
Azure Front Door Service (AFD) es un punto de entrada escalable de alcance global que utiliza nuestro perímetro de red inteligente para ofrecer la posibilidad de crear aplicaciones web rápidas, seguras y de gran escalabilidad.  Debido a que se ha diseñado para ofrecer compatibilidad con las cargas de trabajo web más grandes, entre otras, Bing, Office 365, Xbox Live, MSN y Azure DevOps, AFD ofrece una confiabilidad y escalabilidad a escala Web.

![AFD_Esquema]({{ site.baseurl }}/images/AFD_Esquema.png)

AFD, que actualmente está disponible en 33 países en las ubicaciones de red de Microsoft conectadas mediante nuestra WAN global, mejora el rendimiento de la aplicación gracias a su aceleración, a la descarga de SSL, a que permite enrutar el tráfico HTTP global al back-end más próximo disponible y a que ofrece una confiabilidad de nivel empresarial con una conmutación por error instantánea y automática.

Con una ruta de AFD que conozca el enrutamiento, el almacenamiento en caché en línea, la limitación de frecuencia y la seguridad en el nivel de la aplicación, puede compilar aplicaciones modernas y globales en Azure.  Un panel y un plano de control centralizados permite administrar y controlar el tráfico del servicio y los back-ends de microservicios globales dentro y fuera de Azure.

La integración de AFD con Azure Web Apps, Azure Monitor y Log Analytics permite acelerar y entregar las aplicaciones fácilmente con una latencia más baja, mayor confiabilidad e información sobre tráfico global más pormenorizada. Consulte la documentación sobre AFD para obtener información sobre cómo acelerar su aplicación.

En resumen, Azure Front Door es una combinacion de las siguientes herramientas que ya nos ofrecia Azure **(Traffic Manager, Application Gateway, Load Balancer y Azure CDN)**, potenciandolas en una sola herramienta.

Las caracteristicas nos ofrece Azure Front Door son las siguientes:

-  La aceleración de aplicaciones con difusión por proximidad y el uso de la red global privada masiva de Microsoft para conectarlo directamente a los back-ends implementados de Azure permiten la ejecución de la aplicación con menor latencia y un rendimiento más alto para los usuarios finales. 

- El equilibrio de carga HTTP le permite crear aplicaciones de forma resistente en todas las regiones, conmutar por error instantáneamente y ofrecer a los usuarios una experiencia de disponibilidad de sitio web "AlwaysOn".  

- El enrutamiento basado en la ruta de acceso impulsa las aplicaciones de microservicios globales con enrutamiento independiente en todo el dominio único global. 

- Un único panel de vidrio para supervisar el tráfico del usuario y el mantenimiento del servicio de back-end distribuido y obtener información acerca de esto.

- El protocolo IPv6, los certificados SSL personalizados, la limitación de la velocidad y una infinidad de características le proporcionan la capacidad de personalización para poder adaptarse a las necesidades básicas de la aplicación.

En una proxima edicion les estare compartiendo un video tutorial sobre como funciona Azure Front Door.

### Links de interés: ###

[Azure Front Door][Azure Front Door]

[Azure Front Door]: https://azure.microsoft.com/en-us/services/frontdoor/
