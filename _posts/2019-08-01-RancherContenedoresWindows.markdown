---
title:  "Rancher - Contenedores en nodos Windows y Linux"
date:   2019-08-01 20:00:00
categories: [Containers]
tags: [Rancher, Windows, Kubernetes, Rancher]
---
Hace poco tiempo que ya contamos con la posibilidad de correr contenedores windows en kubernetes. En esta ocacion se me ocurrio investigar la posibilidad de desplegar algunos contenedores con IIS, que corran aplicaciones .NET y algunos contenedores con SQL. Les voy a ir adelantando que esta va a ser una serie de articulos en la cual les voy a contar como usar Rancher para administrar nodos linux y windows sobre kubernetes y desplegar dichos contenedores.



## Introducción ##

Rancher Labs ya tiene disponible (en su version exprimental), la posibilidad de administrar nodos windows bajo algunas condiciones, para ello se requiere tener la version **2.3.0** de Rancher.

**De momento esta version esta en version exprimental y Rancher Labs no la recomienda para implementar en ambientes de produccion.**

A continuacion vamos a aprovisionar un mix de nodos windows y linux, que formaran nuestro cluster de kubernetes administrado via Rancher.


## Pre-requisitos ##

Antes de aprovisionar el cluster, debemos asegurarnos de que tenemos todos los agentes de rancher instalados en cada nocdo y que todo el trafico de red esta habilitado para que el master de rancher pueda administrar los nodos.

## Requisitos para el nodo ##

Para poder agregar nodos Windows se requiere la version de Windows Server 2019 core. (Ej: Windows Server Core 2019 1809 o superior), las versiones anteriores no son soportadas.

## Requisitos para los contenedores ##

Si contamos con contenedores que fueron desplegados en versiones anteriores de Windows Server 2019 1809, y queremos desplegarlos en este cluster, vamos a tener que redesplegar los mismos.

## Pasos para desplegar contenedores Windows ##

Para desplegar nodos y contenedores que soporten Windows debemos seguir los siguientes puntos de la lista:

1- Provisionamiento de hosts.

2- Crear el cluster.

3- Agregar el nodo master en Linux.

4- Agregar el nodo worker en Linux.

5- Agregar los nodos worker en Windows.

6- Configurar las rutas si usamos nodos en Azure.(Host Gateway Mode - Opcional).

7- Configuracion de Azure Files (Opcional).

## 1. Provisionamiento de hosts ##

Para comenzar a aprovisionar un cluster con soporte Windows debemos configurar los siguientes hosts. Este ejemplo contara con tres nodos, dos Linux y un Windows.

En la siguiente tabla les voy a detallar que roles va a complir cada nodo. A continuacion les paso a contar. El primer nodo Linux, es el nodo primario, responsable de la administracion de Kubernetes control plane. El segundo nodo Linux, es un nodo secundario, que tendra el worker, correra el servidor DNS y el Ingress Controller, las metricas del servidor y el agente de Rancher. El tercer nodo Windows tendra el worker y correra los contenedores Windows.

|**Nodo**|**Sistema Operativo**|**Feature del cluster**|
|-|-|-|
| Nodo 1 | Linux (Ubuntu Server 18.04)| Control Plane, etcd, Worker |
| Nodo 2 | Linux (Ubuntu Server 18.04) | Worker |
| Nodo 3 | Windows Server Core 2019 (Version 1803) | Worker |

## 2. Creacion del cluster ##

A continuacion vamos a crear el cluster a traves de Rancher:

1- Desde el dashboard de Rancher, click en la pestaña **Clusters** y seleccionar **Add Cluster**.

2- Primero se nos preguntara en donde va a estar hosteado el cluster, debemos seleccionar **Custom**,

3- Debemos asignar un nombre para el cluster.

4- Vamos a asignar los permisos yendo a la opcion **Member Roles**. Para ello haremos lo siguiente:

	a- Click en **Add Member** y seleccionamos los usuarios que tendran acceso al cluster.

	b- Usaremos la opcion **Role** para asignar los diferentes permisos a los usuarios.

5- Dentro de **Cluster Options** elgigiremos que version de Kubernetes se usara, asimismo que proveedor de red usaremos. 

Para el caso de los nodos Windows, debemos seleccionar las siguientes opciones recomendadas:

- Se debera seleccionar la version 1.14 o superior de Kubernetes.
- Se debera seleccionar el proveedor de red **Flannel**. Dentro de este deberemos seleccionar el modo **VXLAN (Overlay).**
- Se debera habilitar el soporte para nodos Windows.

6- Si los nodos estan hosteados en un proveedor de nube, y buscamos como configurar balanceadores, storage persistentes, etc. debemos seguir la siguente documentacion. 

[Cloud Providers][Cloud Providers]

[Cloud Providers]: https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/rke-clusters/options/cloud-providers/

7- Pro ultimo haremos click en **Next**.

***Importante***

***Si usamos el modo **Host Gateway (L2bridge) y tenemos los nodos hosteados en los siguientes proveedores de nube, deberemos desactivar **private IP address check** del inicio de los nodos Windows y Linux. Para ello seguiremos la recomendacion de la siguiente tabla, de acuerdo al proveedor que usemos.***

|**Servicio**|**Desabilitar private IP address check**|
|-|-|
|Amazon EC2| https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck|
|Google GCE| https://cloud.google.com/vpc/docs/using-routes#canipforward|
|Azure VM| https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface#enable-or-disable-ip-forwarding|

## 3. Agregando el nodo master Linux ##

El primer nodo debera ser Linux el cual debera alojar los roles de **Control Plane** y **etcd**. Antes de alojar el tercer nodo Windows, deberemos tener instalados y configurados los dos nodos Linux.
Siempre se recomienda un minimo de tres nodos y mas.

En la siguiente tabla se mostrara los roles del primer nodo master Linux.

|**Opcion**|**Configuracion**|
|-|-|
|Sistema operativo del nodo|Linux|
|Roles del nodo|etcd, control plane, worker (opcional)|

1- Para el sistema operativo del nodo, seleccionar **Linux**.

2- Desde los roles del nodo, debemos seleccionar la ultima version de **etcd** y **control plane**.

3- Opcional, desde las opciones avanzadas del nodo, podremos selleccionar la configuracion IP, el hostname y las etiquetas.

4- Desplegara un comando que debemos copiar.

5- Conectarse al host linux via ssh, pegar y ejecutar el comando.

6- Una vez que quedo aprovisionado el host, se debe seleccionar **Done**.

**Resultado:**

- El cluster sera creado y tendra el estado de **Provisioning**.
- Podremos acceder al cluster luego de que el estado figure como **Active**.
- El cluster activo sera asignado a dos proyectos, el **Default** (contendra el namespace por defecto) y el **system** (contendra los siguientes namespaces, **cattle-system**, **ingress-nginx**, **kube-public** y **kube-system**).

## 4. Agregando el nodo worker Linux ##

Luego del aprovisionamiento inicial del cluster, nuestro cluster solo tiene un host Linux. Para agregar otro host Linux, el cual sera usado para tener un agente de rancher, las metricas del servidor, DNS e Ingress del cluster. Debemos hacer lo siguiente:

1- Dentro del menu, abrir el cluster creado.

2- Dentro del menu principal del cluster, seleccionar **nodes**.

3- Dentro de la opcion **Node operating system**, seleccionar **Linux**.

4- Sleeccionar el rol **worker**.

5- Desplegara un comando que debemos copiar.

6- Conectarse al host linux via ssh, pegar y ejecutar el comando.

7- Desde la consola de Rancher, click en **Save**.

**Resultado:**

El **worker** esta instalado en el nodo Linux y registrado en Rancher.

## 5. Agregando el nodo worker Windows ##

Podemos agregar host Windows al cluster, editando el cluster y seleccionando la opcion Windows.

1- Desde el menu principal, seleccionar **Nodes**.

2- Click en **Edit Cluster**.

3- Dentro de la opcion **Node operating system**, seleccionar **Windows**.

4- Desplegara un comando que debemos copiar.

5- Conectarse al hosr Windows via RDP, abrir una consola de CMD, pegar y ejecutar el comando.

6- Desde la consola de Rancher, click en **Save**.

7- Opcional: Repetir los mismos pasos para agregar mas nodos Windows al cluster.

**Resultado:**

El **worker** esta instalado en el nodo Windows y registrado en Rancher.




