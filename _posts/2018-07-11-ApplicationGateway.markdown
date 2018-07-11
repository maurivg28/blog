---
title:  "Application Gateway"
date:   2018-07-11 14:15:00
categories: [Ansible]
tags: [ApplicationGateway, Azure, Cloud, Microsoft, DevOps]
---
Vamos a partir de la base, la cual debemos utilizar un equilibrador de carga para nuestra solucion web. Lo primero que se nos viene a nuestra mente es un balanceador, pero Azure nos ofrece un servicio mucho mas flexible, seguro, el cual nos ofrece todas las caracteristicas necesarias para exponer de manera segura nuestra aplicacion web.
Hoy vamos a hablar de Azure Application Gateway.


## Que nos ofrece Azure Application Gateway? ##

[ * ] ***Servicio escalable y de alta disponibilidad.***
Application gateway nos proporciona un servicio de equilibrio de carga y enrutamiento a nivel de aplicacion que permiten crear sistemas front-end web escalables y de alta disponibilidad en Azure.

[ * ] ***Web Application Firewall.***
Nos ofrece un servicio de firewall que proteje a nuestras aplicaciones ante vulnerabilidades web y de seguridad habituales como inyección de código SQL o filtros entre sitios.

[ * ] ***Front-end web eficiente y seguro.***
Permite manejar certificados lo cual garantiza un front end web mucho mas seguro.

[ * ] ***Perfecta integración con los servicios de Azure***
Application Gateway facilita la integración con Azure Traffic Manager para permitir el redireccionamiento entre varias regiones, la conmutación por error automática y el mantenimiento sin tiempo de inactividad. Application Gateway se integra también con Azure Load Balancer para ofrecer escalado horizontal y alta disponibilidad de sistemas front-end web, tanto internos como para Internet.

## Que es? ##

Application Gateway es un balanceador web capaz que administrar el trafico hacia las aplicaciones web.

A diferencia de los balanceadores tradicionales que trabajan a nivel de la capa de transporte (Capa 4 TCP y UDP),  ruteaando el trafico basado en una ip de origen y puerto, hacia una ip y puerto de destino. 
Application Gateway nos permite ser mas especificos. Por ejemplo, podemos rutear trafico hacia una URL especifica. Entonces, si los usuarios quieren acceder a ***/images***, podremos rutear dicho trafico a un pool de servidores configurados especificamente para contener imagenes. Si los usuarios quieren acceder a ***/video***, podemos rutear dicho trafico a un pool de servidores epecificos, optimizados para videos.

![Application_GatewayEsquema]({{ site.baseurl }}/images/ApplicationGW_Esquema.png)

Application Gateway trabaja a nivel de la capa de aplicacion (Capa 7). Ademas de realizar ruteo basado en URL, tenemos otras caracteristicas que nos ofrece dicho servicio:

## Ruteo basado en URL ##

El ruteo a nivel de URL permite rutear el trafico hacia el pools servidores back-end basado en la URL o el Path hacia donde el usuario quiere acceder.
Uno de los escenarios mas comunes es el ruteo del trafico hacia diferentes poll's de servidores back-end.

Por ejemplo, todos los request que son para ```http://blog.mvillagran.com/video/*``` seran ruteados hacia los pools de servidores de Video, y los request para ```http://blog.mvillagran.com/images/*``` seran ruteados hacia los pools de servidores de Imagenes. Todos los demas request que no machean con las anteriores iran a una regla por defecto.


### Redireccion ###

Un escenario muy comun es cuando tenemos multiples aplicaciones web, las cuales atienden diferentes URL y ademas atienden tanto en HTTP como en HTTP, y queremos que todo el trafico sea ruteado por un camino seguro y encriptado.
Con Application Gateway no solo podemos redireccionar trafico HTTP hacia HTTPS sino que tambien tiene otras caracteristicas como:

[ * ] Redireccion global desde un puerto hacia otro puerto, claro ejemplo que mencionaba anteriormente HTTP hacia HTTPS.

[ * ] Redireccion hacia un path, por ejemplo no solo podemos redireccionar un sitio de HTTP hacia HTTPS, sino que tambien a un area especifica del sitio como ser ```/services/*```

[ * ] Reddireccion hacia un sitio externo.

### Multiple Site Hosting ###

Application Gateway, permite alojar varios sitios diferentes en una misma instancia del servicio. Permite que podamos configurar hasta 20 sitios en una misma instancia. Por ejemplo, podemos alojar dos servicios diferentes, uno ```http://autos.com``` y otro ```http://motos.com``` y cada uno de ellos corresponde a un pools de servidores back-end diferentes.

Los requests para ```http://autos.com``` seran ruteados hacia el pool de servidores back-end ***AutosServerPool***, y los request para ```http://motos.com``` seran ruteados hacia el pool ***MotosServerPool***.

### Session Affinity ###

La sesion basada en cookie es muy util cuando se quiere mantener dicha sesion de usuario en un mismo servidor. Usando gateway-managed cookies, Appliation Gateway puede direccionar el trafico para una sesion de usuario hacia el mismo servidor que esta procesando dicha solicitud.

### Secure Sockets Layer (SSL) termination ###

Application Gateway es capaz de encriptar y desencriptar el trafico SSL, de manera que el trafico que va hacia los servidores de back-end no vaya encriptado y de esta manera los servidores back-end no se vean sobrecargados.
A veces en las empresas las comunicaciones no encriptadas, no es una opcion valida por lo cual Application gateway tambien puede encriptar el trafico hacia los servidores back-end en caso de ser necesario.

### Web application Firewall ###

Web application firewall (WAF) es una feature que contamos con este servicio, la cual nos provee proteccion centralizada contra ataques.

WAF nos proteje de:

[ * ] Ataques usados con tecnicas de exploits.

[ * ] SQL Injection.

[ * ] Cross site scripting.

[ * ] Ataques al codigo de la aplicacion.

Es una caracteristica la cual debemos despreocuparnos ya que no requiere administracion, todos los parches, monitoreo y reglas corre por cuenta de Microsoft. Esto nos garantiza y no da la tranquilidad de que nuestras aplicaciones estaran protegidas. 

### Websocket and HTTP/2 traffic ###

Application Gateway ofrece soporte nativo para los protocolos WebSocket y HTTP/2. Vale la pena comentar que esta caracteristica solo la podremos habilitar/deshabilitar a traves de Powershell.

En un proximo capitulo les voy a contar como configurar este servicio y les voy a dar algunos tips ya que la configuracion del mismo no es tan trivial como parece ;D
