---
title:  "Kubernetes"
date:   2019-01-02 20:00:00
categories: [Kubernetes]
tags: [Kubernetes, Containers, Docker]
---
En este articulo pretendo hablar de manera ordenada sobre los conceptos basicos que hacen de kubernetes el orquestador de contenedores mas usado para orquestar aplicaciones que corren sobre contenedores.


## Conceptos a tener en cuenta ##

- **Cluster:** Conjunto de máquinas físicas o virtuales que son utilizados por Kubernetes
- **Pod:** es la unidad mínima de Kubernetes, realmente es un contenedor en jerga de Docker
- **Labels y selectors:** son pares de claves y valores, las cuales se pueden aplicar a pods, services, replication controllers, etc…. y con ellos podremos identificarlos para poderlos gestionar.
- **Node:** es el servidor ya sea virtual o físico que aloja el sistema de Kubernetes y donde vamos a desplegar nuestros pods (contenedores). Si buscáis información por internet, antiguamente se llamaban Minions.
- **Replication Controller:** es el responsable de gestionar la vida de los pods y el encargado de mantener arrancados los pods que se le hayan indicado en la configuración. Permite escalar de forma muy sencilla los sistemas y maneja la recreación de un pod cuando ocurre algún tipo de fallo.
- **Replica Sets:** Es la nueva generación del Replication Controller, con nuevas funcionalidades. Una de las funcionalidades destacadas es que nos permite desplegar pods en función de los labels y selectors
- **Deployments:** Es donde e especifican la cantidad de réplicas de pods que tendremos en el sistema. Es una funcionalidad más avanzada que los Replication Controller y muy parecida a los Replciation Sets, pero con otras características.
- **Namespaces:** son agrupaciones, en ellos podremos diferenciar espacios de trabajo para diferentes situaciones. Por ejemplo podríamos realizar un Namespace para producción y otro para desarrollo y cada Namespace tendría sus propios pods, replication controllers, etc….
- **Volumes:** Es el acceso a un sistema de almacenamiento
- **Secrets:** Es donde se guarda la información confidencial como usuarios y passwords, para poder acceder a los recursos.
- **Service:** Es la política de acceso a los pods. Lo podríamos definir como la abstracción que define un conjunto de pods y la lógica para poder acceder a ellos.

## Que es kubernetes? ##

Basicamente vamos a comenzar por una corta definicion y vamos a quedarnos con el concepto de "orquestador de contenedores"

**Definicion**
Kubernetes (referido en inglés comúnmente como “K8s”) es un sistema de código libre para la automatización del despliegue, ajuste de escala y manejo de aplicaciones en contenedores​ que fue originalmente diseñado por Google y donado a la Cloud Native Computing Foundation (parte de la Linux Foundation).

**Un cluster de kubernetes esta formado por dos tipos de recursos:**

- **Master** es el encargado de manejar el cluster. Coordina todas las actividades en su clúster, como programar aplicaciones, mantener el estado deseado de las aplicaciones, escalar aplicaciones y desplegar nuevas actualizaciones.
- **Nodes** es el servidor ya sea virtual o físico que aloja el sistema de Kubernetes y donde vamos a desplegar nuestros pods (contenedores). 
Cada nodo tiene un **Kubelet**, que es un agente para administrar el nodo y comunicarse con el maestro Kubernetes. El nodo también debe tener herramientas para manejar las operaciones de contenedores, como **Docker o rkt**. Un clúster de Kubernetes que maneje el tráfico de producción debe tener un **mínimo de tres nodos.**

Cuando implementan aplicaciones en Kubernetes, se le envia una orden al **master** que inicie los contenedores de aplicaciones. El **master** programa los contenedores para que se ejecuten en los nodos del clúster. Los nodos se comunican con el maestro utilizando la API de Kubernetes, que el **master** expone.

**Diagrama**
![k8s_Diagram]({{ site.baseurl }}/images/k8s_cluster_diagram.png)

Ahora vamos a ver algunas herramientas que forman parte de kubernetes:

- **minikube** para realizar pruebas se creó Minikube, el cual es una versión reducida de Kubernetes, corriendo en una única máquina virtual, la cual  actúa de maestro y esclavos a la vez.

- **kubectl** es necesario para poder comunicarse con el servidor Kubernetes.

Algunos comandos utiles con kubectl:

Si queremos ver la informacion del cluster: `kubectl cluster-info`

```sh
kubectl cluster-info
Kubernetes master is running at https://172.17.0.59:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Si queremos ver todos los nodos que forman parte del cluster: `kubectl get nodes`

```sh
$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    14m       v1.10.0
```

## Desplegando una aplicacion ##

Una vez que tenemos el cluster corriendo, podremos desplegar contenedores dentro del cluster, para ello debemos crear una configuracion de despliegue **(Deployment Configuration)**. Cuando se ejecuta un deployment de configuracion, se le indica a kubernetes que debe crear y actualizar las instancias de nuestra aplicacion. Una vez que esta creado el despliegue, el master de kubernetes programa las instancias que deben correr en cada nodo miembro del cluster.

Una vez que la aplicacion ya esta creada y corriendo, Kubernetes **Deployment Controller**, monitorea continuamente estas instancias, si un nodo se cae o es borrado, el **Deployment Controller** enseguida lo remplaza.

![k8s_App_Diagram]({{ site.baseurl }}/images/k8s_cluster_app_diagram.png)

Podremos crear y administrar un despiegue (**deployment**) usando **kubectl**. Kubectl utiliza la API para interactuar con el cluster.

A continuacion vamos a desplegar una aplicacion de ejemplo sobre el cluster:

1- Desplegamos la aplicacion con el siguiente comando:

```sh
$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
deployment.apps/kubernetes-bootcamp created
```
La aplicacion se despliega ejecutando `kubectl run`, dicho comando crea una nueva implementacion. A dicho comando se le proporciona un nombre y la ubicacion de la imagen de la aplicacion (en este caso se le especifica la URL del repositorio de docker). Luego le pasamos el parametro `--port:` para especificarle el puerto al que queremos exponer nuestra aplicacion.

2- Con el siguiente comando, verificamos el correcto despliegue:

```sh
kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           4m
```

3- Visualizando la appliaccion. 

Los **Pods** (pensemos que el POD es un contenedor) corren dentro de Kubernetes y no estan expuestos al mundo. Estan aislados en una red privada. Por defecto son visibles desde otros Pods o servicios dentro del mismo cluster, pero no son accesibles desde fuera de la red.
La manera de interactuar con estos Pods, es a traves de la herramienta kubectl, la cual interactua con la API de kubernetes.

Podemos establecer una conexion entre el host y el cluster de kubernetes a traves de un proxy.
Para ello ejecutamos `kubectl proxy`. El proxy permite el acceso directo a la API desde estos terminales.

```sh
kubectl proxy
$
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```
Luego podremos ver todas las APIs alojadas a traves del proxy, ejecutando `curl http://localhost:8001/version`

```sh
curl http://localhost:8001/version
{
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.0",
  "gitCommit": "fc32d2f3698e36b93322a3465f63a14e9f0eaead",
  "gitTreeState": "clean",
  "buildDate": "2018-04-10T12:46:31Z",
  "goVersion": "go1.9.4",
  "compiler": "gc",
  "platform": "linux/amd64"
}$
```
La API creará automáticamente una conexion para cada pod, según el nombre del pod, al que también se puede acceder a través del proxy.

Primero necesitamos saber el nombre del POD, dicho nombre esta alojado en la variable de entorno llamada POD_NAME:

```sh
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metaata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5c69669756-52r9w
```

Ahora vamos a probar la aplicacion que se esta ejecutando en este POD, realizando una solicitud HTTP:

```sh
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-52r9w | v=1
```
## Pods y Nodos ##

**Kubernetes Pods**

Un POD es la instancia mínima  que usará Kubernetes consistirá de al menos una imagen Docker, aunque pueden ser más. Suele usarse una sola por facilidad de control,  por lo tanto,  un pod puede ser desde una aplicación SpringBoot (sirviendo una REST API).

Los pods por definición son **stateless**,  Kubernetes los desplegará y destruirá constantemente  en función de las necesidades  actuales. Si los pods deben persistir datos,  deben apoyarse en volumes compartidos como por ejemplo NFS.

Cuando creamos una Deplyment en Kubernetes, esa deplyment crea Pods con contenedores dentro de ellos (en lugar de crear contenedores directamente). Cada Pod está vinculado al Nodo donde está programado y permanece allí hasta la terminación (según la política de reinicio) o la eliminación. En caso de una falla de Nodo, los Pods idénticos están programados en otros Nodos disponibles en el clúster.

![k8s_Pods_Diagram]({{ site.baseurl }}/images/k8s_cluster_pods_diagram.png)

**Nodos**

Un **Pod** corre en un nodo. Un nodo es una maquina llamada **worker** que podria ser una maquina virtual o fisica. Cada nodo es administrado por el **Master**. Un nodo puede tener multiples pods, y el **master** maneja automaticamente la programacion de los nodos a traves del cluster. La programacion es automatica y tiene en cuenta los recursos disponibles en cada nodo.

Cada nodo de kubernetes ejecuta al menos:

- **kublet** es el proceso responsable de la comunicacion entre el **master** y el nodo. Tambien gestiona los pods y los contenedores que se ejecutan en cada maquina.

- **container runtime** (Doker, rkt), es el responsable del pull de las imagenes desde los repositorios (docker registry), tambien de desempaquetar el contenedor y correr la aplicacion.

![k8s_node_Diagram]({{ site.baseurl }}/images/k8s_cluster_node_diagram.png)

## Comandos utiles para troubleshooting ##

- **kubectl get** lista los recursos
- **kubectl describe** da informacion detallada acerca de los recursos
- **kubectl logs** imprime los logs de un contenedor en un pod
- **kubectl exec** ejecuta un comando de un contenedor en un pod

A continuacion ejecutaremos los comandos mencionados anteriormente para ver como funcionan:

**kubectl describe**

```sh
$ kubectl describe pods
Name:           kubernetes-bootcamp-5c69669756-nz2gp
Namespace:      default
Node:           minikube/172.17.0.16
Start Time:     Thu, 03 Jan 2019 01:48:45 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://f085623a9866cf52e740bb1414bdae6280d6912e7920715653be5607334ab72e
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 01:48:46 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gdhfp (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-gdhfp:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gdhfp
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age                From               Message
  ----     ------                 ----               ----               -------
  Warning  FailedScheduling       52s (x4 over 55s)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              48s                default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-nz2gp to minikube
  Normal   SuccessfulMountVolume  48s                kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-gdhfp"
  Normal   Pulled                 47s                kubelet, minikube  Container image"gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                47s                kubelet, minikube  Created container
  Normal   Started                47s                kubelet, minikube  Started container
```
**kubectl logs**

```sh
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5c69669756-c5m9d
$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-c5m9d | v=1
$
$
$ kubectl logs $POD_NAME
Kubernetes Bootcamp App Started At: 2019-01-03T01:51:35.933Z | Running On:  kubernetes-bootcamp-5c69669756-c5m9d

Running On: kubernetes-bootcamp-5c69669756-c5m9d | Total Requests: 1 | App Uptime: 37.56 seconds | Log Time: 2019-01-03T01:52:13.493Z
```
**kubectl exec**

```sh
 kubectl exec $POD_NAME env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-5c69669756-c5m9d
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root

 kubectl exec -ti $POD_NAME bash
root@kubernetes-bootcamp-5c69669756-c5m9d:/#

root@kubernetes-bootcamp-5c69669756-c5m9d:/# cat server.js
var http = require('http');
var requests=0;
var podname= process.env.HOSTNAME;
var startTime;
var host;
var handleRequest = function(request, response) {
  response.setHeader('Content-Type', 'text/plain');
  response.writeHead(200);
  response.write("Hello Kubernetes bootcamp! | Running on: ");
  response.write(host);
  response.end(" | v=1\n");
  console.log("Running On:" ,host, "| Total Requests:", ++requests,"| App Uptime:", (new Date() - startTime)/1000 , "seconds", "| Log Time:",new Date());
}
var www = http.createServer(handleRequest);
www.listen(8080,function () {
    startTime = new Date();;
    host = process.env.HOSTNAME;
    console.log ("Kubernetes Bootcamp App Started At:",startTime, "| Running On: " ,host, "\n" );
});

root@kubernetes-bootcamp-5c69669756-c5m9d:/# curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-c5m9d | v=1

exit
```

## Exponiendo la aplicacion ##

Los Pods de kubernetes son "mortales", esto quiere decir que tienen un ciclo de vida. Cuando un nodo muere, todos los pods que corren en ese nodo se pierden. A continuacion el **Replicaset** (Nueva generacion de Replication Controller) crea dinamicamente nuevos pods para que la aplicacion siga corriendo.


**Ejemplo**: condiderando que tenemos una aplicacion con 3 repicas. Esas replicas son intercambiables, para el usuario final es transparente si cambian dichas replicas o incluso si muere un pod y el mismo es recreado.

Dicho esto, cada Pod en un clúster Kubernetes tiene una dirección IP única, incluso los Pods en el mismo Nodo, por lo que debe haber una forma de reconciliar automáticamente los cambios entre Pods para que las aplicaciones continúen funcionando.

Como habiamos dicho anteriormente. Un servicio en Kubernetes es una abstracción que define un conjunto lógico de Pods y una política mediante la cual acceder a ellos. Los servicios permiten un acoplamiento suelto entre Pods dependientes. Un servicio se define utilizando YAML (preferido) o JSON, como todos los objetos Kubernetes. El conjunto de Pods a los que apunta un Servicio suele estar determinado por un LabelSelector.

Aunque cada Pod tiene una dirección IP única, esas IP no están expuestas fuera del clúster sin un Servicio. Los servicios permiten que las aplicaciones reciban tráfico. Los servicios se pueden exponer de diferentes maneras especificando un tipo en la configuracion del **ServiceSpec**:


- **CusterIP(default): Expone el servicio con una IP interna dentro del cluster. Esto hace que el servicio quede accesible desde dentro del cluster.

- **NodePort**: Expone el servicio en el mismo puerto de cada Nodo seleccionado en el cluster usando NAT. Esto hace que un servicio sea accedible desde fuera del cluster usando `<NodeIP>:<NodePort>`

- **LoadBalancer**: Crea un balanceador externo del cluster actual y asigna una ip externa fija al servicio.

- **ExternalName**: Expone el servicio usando un nombre aleatorio (especificado en el atributo `externalName`) devolviendo un registro CNAME con el nombre. Esto requiere la version 1.7 de **kube-dns** o superior.

![k8s_service_Diagram]({{ site.baseurl }}/images/k8s_cluster_service_diagram.png)

El servicio rutea el trafico a traves del conjunto de Pods. Los servicios son una abstraccion que permite si un Pod muere se generen otras replicas en Kubernetes sin impactar en la aplicacion que esta corriendo.

El descubrimiento y enrutamiento entre los Pods dependientes (como los componentes de frontend y backend en una aplicación) son manejados por los Servicios de Kubernetes.

Un servicio coincide con una grupo de Pods que usan **labels and selectors**, es una agrupacion logia que permite a Kubernetes operar con estos objetos. Los **labels** son un par **key/value** que esta en cada objeto y se puede usar de varias maneras:

- Designar objetos para ambientes (development, test y production).

- Tags de version.

- Clasificar los objetos.

![k8s_tags_Diagram]({{ site.baseurl }}/images/k8s_cluster_tags_diagram.png)

A continuacion vamos a ver un ejemplo practico de lo que estuvimos hablando anteriormente:

1- Veremos los pods que estan corriendo con el siguiente comando:

```sh
kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-9qrl2   1/1       Running   0          1m
```

2- Listaremos los servicios que estan corriendo en el cluster:

```sh
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m
```

Aqui tenemos un servicio llamado **kubernetes** que se crea cuando cuando minikube inicia el cluster. A conotinuacion vamos a crear otro servicio y exponerlo externamente, para exponerlo, en el comando le pasamos el parametro **NodePort**

3- Vamos a crear un nuevo servicio y exponerlo:

```sh
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service/kubernetes-bootcamp exposed
```

4- Verificamos que el servicio se haya creado:

```sh
kubectl get services
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP          6m
kubernetes-bootcamp   NodePort    10.107.112.197   <none>        8080:30225/TCP   38s
```

Aqui tenemos el servicio llamado **kubernetes-bootcamp**. Vemos que el Servicio recibió una IP de clúster única, un puerto interno y una IP externa (la IP del Nodo).

5- Para ver qué puerto se abrió externamente (mediante la opción NodePort), ejecutaremos el comando `describe service`:

```sh
 kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.107.112.197
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30225/TCP
Endpoints:                172.18.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$
```

6- Vamos a crear una variable de entorno llamada NODE_PORT que tenga el valor del puerto de nodo asignado:

```sh
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=30225
```

7- Ahora podemos probar que la aplicación está expuesta fuera del clúster utilizando curl, la IP del nodo y el puerto expuesto externamente:

```sh
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-9qrl2 | v=1
```

8- El deployment creó automáticamente una etiqueta para nuestro Pod. Con el comando `describe deployment` puede ver el nombre de la etiqueta:

```sh
 kubectl describe deployment
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Thu, 03 Jan 2019 13:26:31 +0000
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=kubernetes-bootcamp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:        gcr.io/google-samples/kubernetes-bootcamp:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-5c69669756 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  14m   deployment-controller  Scaled up replica set kubernetes-bootcamp-5c69669756 to 1
```

9- Usaremos el tag para consultar la lista de pods:

```sh
kubectl get pods -l run=kubernetes-bootcamp
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-9qrl2   1/1       Running   0          18m
```

10- Podemos hacer lo mismo para listar los servicios existentes:

```sh
$ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.107.112.197   <none>        8080:30225/TCP   14m
```

11- Vamos a obtener el nombre del Pod y guardarlo en una variable llamada **POD_Name**:

```sh
 export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5c69669756-9qrl2
```

12- Vamos a  aplicar una nueva etiqueta, usamos el comando label seguido del tipo de objeto, el nombre del objeto y la nueva etiqueta:

```sh
$ kubectl label pod $POD_NAME app=v1
pod/kubernetes-bootcamp-5c69669756-9qrl2 labeled
```

13- Ejecutando el comando `describe pods` podremos ver con mas detalle la etiqueta que quedo en nuestro Pod:

```sh
$ kubectl describe pods $POD_NAME
Name:           kubernetes-bootcamp-5c69669756-9qrl2
Namespace:      default
Node:           minikube/172.17.0.115
Start Time:     Thu, 03 Jan 2019 13:26:38 +0000
Labels:         app=v1
                pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.4
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://029f41eef268db44a5942fd4abe2f6736c760fd2356d903257de9a69cfcaf66b
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 13:26:41 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rhgf4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-rhgf4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-rhgf4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age                From               Message
  ----     ------                 ----               ----               -------
  Warning  FailedScheduling       25m (x4 over 25m)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              25m                default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-9qrl2 to minikube
  Normal   SuccessfulMountVolume  25m                kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-rhgf4"
  Normal   Pulled                 25m                kubelet, minikube  Container image"gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                25m                kubelet, minikube  Created container
  Normal   Started                25m                kubelet, minikube  Started container
```

14- Podremos consultar la lista de Pods, consultando a traves de la nueva etiqueta:

```sh
$ kubectl get pods -l app=v1
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-9qrl2   1/1       Running   0          27m
```

15- Podremos borrar el servicio creado usando el comando `delete service`:

```sh
 kubectl delete service -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted
```

16- Vamos a confirmar que el servicio fue borrado:

```sh
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   35m
```

17- Ahora vamos a comprobar que no quedo nada expuesto:

```sh
 curl $(minikube ip):$NODE_PORT
curl: (7) Failed to connect to 172.17.0.115 port 30225: Connection refused
```

18- Tambien vamos a comprobar que la app ya no esta expuesta y solo se ejecuta dentro del Pod:

```sh
$ kubectl exec -ti $POD_NAME curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-9qrl2 | v=1
```

## Corriendo multiples instancias ##

**Escalando la aplicacion**

Anteriormente hemos creado un **Deployment**, y fue expuesto a traves de un **Service**. En el Deployment solamente se ha creado un Pod en donde corria la aplicacion. Cuando el trafico se incrementa, necesitamos escalar la aplicacion para no afectar a los usuarios que consumen la misma.

![k8s_scaling1_Diagram]({{ site.baseurl }}/images/k8s_cluster_scaling1_diagram.png)

![k8s_scaling2_Diagram]({{ site.baseurl }}/images/k8s_cluster_scaling2_diagram.png)

El escalameinto del **Deployment** nos asegura que se crean nuevos Pods y se programan nuevos nodos para tener siempre recursos disponibles. El escalamiento aumentara el numero de pods al nuevo estado deseado. Kubernetes permite el autoescalado automatico de Pods. Tambien se puede el escalamiento "hacia abajo", el cual termina los pods del Deployment.

Correr multiples instancias de una aplicacion va a requerir que todo el trafico sea distribuido. Los **Services** tienen integrado un **load-balancer** que se encarga de distribuir el trafico de red entre todos los pods que estan expuestos en el **Deployment**.
El servicio realiza un monitoreo continuo de los enpoints de los Pods, para asegurarse que el trafico solo sea ditribuido a los pods que estan disponibles en el cluster.

Cuando se tienen multiples instancias corriendo, se pueden ejecutar **Rolling Updates** sin downtime.

A continuacion veremos de manera practica lo que hemos estado hablando:

1- Listaremos los deployments que se estan usando:

```sh
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           1m
```
Vamos a explicar la informacion que nos trae el comando:

- **Desired** nos muestra el numero de replicas.
- **Current** nos muestra cuantas replicas tenemos corriendo ahora.
- **Up-To-Date** nos muestra el numero de replicas que estan actualizadas con respecto al estado deseado.
- **Available** nos muestra el numero d replicas disponibles para los usuarios.

2- Vamos a escalar la cantidad de replicas a 4:

```sh
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
deployment.extensions/kubernetes-bootcamp scaled
```

3- Nuevamente listaremos los deployments para ver que se han la cantidad escalado las replicas:

```sh
 kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           7m
```

5- Vamos a chequear si el numero de Pods ha cambiado:

```sh
$ kubectl get pods -o wide
NAME                                   READY     STATUS    RESTARTS   AGE       IP     NODE
kubernetes-bootcamp-5c69669756-bkw2z   1/1       Running   0          1m        172.18.0.5   minikube
kubernetes-bootcamp-5c69669756-hkbj5   1/1       Running   0          1m        172.18.0.7   minikube
kubernetes-bootcamp-5c69669756-jmwwx   1/1       Running   0          8m        172.18.0.2   minikube
kubernetes-bootcamp-5c69669756-vswhc   1/1       Running   0          1m        172.18.0.6   minikube
```

Tenemos 4 Pods con diferentes IPs, el cambio fue resgistrado en el log del Deployment y podemos verlo ejecutando el comando `describe `.

```sh
$ kubectl describe deployments/kubernetes-bootcamp
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Thu, 03 Jan 2019 15:01:36 +0000
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=kubernetes-bootcamp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:        gcr.io/google-samples/kubernetes-bootcamp:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-5c69669756 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set kubernetes-bootcamp-5c69669756 to 1
  Normal  ScalingReplicaSet  3m    deployment-controller  Scaled up replica set kubernetes-bootcamp-5c69669756 to 4
```

6- Vamos a verificar el trafico, puertos expuestos del **load-balancer**:

```sh
$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.109.163.196
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32215/TCP
Endpoints:                172.18.0.2:8080,172.18.0.5:8080,172.18.0.6:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

7- Vamos a crear una variable llamada **NODE_PORT** la cual va a quedar guardado el valor del puerto del nodo:

```sh
 export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=32215
```

8- Ejecutaremos curl contra la ip y el puerto expuesto del balanceador (lo ejecutaremos varias veces para comprobar el que el balanceador esta funcionando correctamente):

```sh
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-bkw2z | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-jmwwx | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-bkw2z | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-vswhc | v=1
```
9- Ahora vamos a ejecutar el comando `scale` para bajar el numero de replicas a 2:

```sh
$ kubectl scale deployments/kubernetes-bootcamp --replicas=2
deployment.extensions/kubernetes-bootcamp scaled
```

10- Vamos a comprobar la cantidad de replicas que tenemos corriendo:

```sh
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   2         2         2            2           20m
```

11- Ahora vamos a comprobar la cantidad de Pods que tenemos corriendo:

```sh
$ kubectl get pods -o wide
NAME                                   READY     STATUS    RESTARTS   AGE       IP     NODE
kubernetes-bootcamp-5c69669756-bkw2z   1/1       Running   0          14m       172.18.0.5   minikube
kubernetes-bootcamp-5c69669756-jmwwx   1/1       Running   0          21m       172.18.0.2   minikube
```
Como podemos ver hay dos Pods que fueron destruidos o terminados.

## Rolling Updates ##

**Ejecutando updates de la aplicacion**

Como sabemos la aplicacion tiene que estar siempre disponible para los usuarios y los desarrolladores esperan que las aplicaciones estan siempre actualizadas.

Cuando se tienen multiples instancias corriendo, se pueden ejecutar **Rolling Updates** sin downtime. **Rolling Updates** permite desplegar la aplicacion actualizada sin downtime incrementando la cantidad de pods ya actualizados. Los nuevos pods son programados en Nodos con recursos disponibles.

![k8s_update1_Diagram]({{ site.baseurl }}/images/k8s_cluster_update1_diagram.png)

![k8s_update2_Diagram]({{ site.baseurl }}/images/k8s_cluster_update2_diagram.png)

![k8s_update3_Diagram]({{ site.baseurl }}/images/k8s_cluster_update3_diagram.png)

![k8s_update4_Diagram]({{ site.baseurl }}/images/k8s_cluster_update4_diagram.png)

Similar al esclado de una aplicacion, el deployment se expone publicamente, a traves del load-balance el trafico solo va hacia los pods que estan disponibles durante la actualizacion.

Durante un **Rolling Update** estan permitidas las siguientes acciones:

- Promover una aplicación de un entorno a otro (a través de actualizaciones de imagen de contenedor)
- Ejecutar un Rollback a versiones previas.
- Ejecutar CI/CD sin perdida.

A continuacion vamos a ejecutar un update de la aplicacion que tenemos corriendo en nuestro cluster:

1- Primero vamos a listar los **Deployments** que tenemos:

```sh
kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           1m
```

2- Luego vamos a listar los Pods que tenemos corriendo:

```sh
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-4lpxw   1/1       Running   0          2m
kubernetes-bootcamp-5c69669756-b666d   1/1       Running   0          2m
kubernetes-bootcamp-5c69669756-ffkhz   1/1       Running   0          2m
kubernetes-bootcamp-5c69669756-tgq47   1/1       Running   0          2m
```

3- Para ver la version de la imagen que esta corriendo en la app ejecutamos:

```sh
$ kubectl describe pods
Name:           kubernetes-bootcamp-5c69669756-4lpxw
Namespace:      default
Node:           minikube/172.17.0.59
Start Time:     Thu, 03 Jan 2019 16:10:01 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.4
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://c0ed5956f375ad6bd7209041c6c3446d0723d803bd06ea39cb9c915a2a2be938
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 16:10:03 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dfmnt (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dfmnt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dfmnt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Warning  FailedScheduling       4m (x3 over 4m)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              4m               default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-4lpxw to minikube
  Normal   SuccessfulMountVolume  4m               kubelet, minikube  MountVolume.SetUpsucceeded for volume "default-token-dfmnt"
  Normal   Pulled                 4m               kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                4m               kubelet, minikube  Created container
  Normal   Started                4m               kubelet, minikube  Started container


Name:           kubernetes-bootcamp-5c69669756-b666d
Namespace:      default
Node:           minikube/172.17.0.59
Start Time:     Thu, 03 Jan 2019 16:10:00 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.3
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://71c78b89fb470877da5643e6562026629835075b622a030e1d2f4de9bc547350
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 16:10:03 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dfmnt (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dfmnt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dfmnt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Warning  FailedScheduling       4m (x3 over 4m)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              4m               default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-b666d to minikube
  Normal   SuccessfulMountVolume  4m               kubelet, minikube  MountVolume.SetUpsucceeded for volume "default-token-dfmnt"
  Normal   Pulled                 4m               kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                4m               kubelet, minikube  Created container
  Normal   Started                4m               kubelet, minikube  Started container


Name:           kubernetes-bootcamp-5c69669756-ffkhz
Namespace:      default
Node:           minikube/172.17.0.59
Start Time:     Thu, 03 Jan 2019 16:09:59 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://824b421f8ea23972e8b8478212608e54435e6a9d02b9482577a128594c4d2d1e
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 16:10:00 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dfmnt (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dfmnt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dfmnt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Warning  FailedScheduling       4m (x2 over 4m)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              4m               default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-ffkhz to minikube
  Normal   SuccessfulMountVolume  4m               kubelet, minikube  MountVolume.SetUpsucceeded for volume "default-token-dfmnt"
  Normal   Pulled                 4m               kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                4m               kubelet, minikube  Created container
  Normal   Started                4m               kubelet, minikube  Started container


Name:           kubernetes-bootcamp-5c69669756-tgq47
Namespace:      default
Node:           minikube/172.17.0.59
Start Time:     Thu, 03 Jan 2019 16:10:01 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.5
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://0e17cb594de67650614e05151604f34ec12d8b78bb582512754557212f255a37
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 16:10:03 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-dfmnt (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-dfmnt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-dfmnt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Warning  FailedScheduling       4m (x3 over 4m)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              4m               default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-tgq47 to minikube
  Normal   SuccessfulMountVolume  4m               kubelet, minikube  MountVolume.SetUpsucceeded for volume "default-token-dfmnt"
  Normal   Pulled                 4m               kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                4m               kubelet, minikube  Created container
  Normal   Started                4m               kubelet, minikube  Started container
```

4- Vamos a cambiar la version de la aplicacion a version 2, usando el comando `set image`:

```sh
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment.extensions/kubernetes-bootcamp image updated
```

El comando que ejecutamos notifica al **Deployment** que use una imagen diferente para la app e inicie el **rolling update**.

5- Vamos a chequear el estatus de los nuevos pods y verificar que los pods con version anterior son terminados:

```sh
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-7799cbcb86-bkdfg   1/1       Running   0          3m
kubernetes-bootcamp-7799cbcb86-jg2sm   1/1       Running   0          3m
kubernetes-bootcamp-7799cbcb86-mqmcm   1/1       Running   0          3m
kubernetes-bootcamp-7799cbcb86-zxcnb   1/1       Running   0          3m
```

6- Ahora vamos a verificar que la app esta corriendo y veremos en que ip/puerto esta expuesta:

```sh
 kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.102.62.254
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32375/TCP
Endpoints:                172.18.0.10:8080,172.18.0.11:8080,172.18.0.8:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

7- Vamos a crear una variable de entorno llamada **NODE_PORT** que tenga el valor del puerto de nodo asignado:

```sh
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=32375
```

8- Vamos a conectarnos a la ip y puerto expuestas:

```sh
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7799cbcb86-jg2sm | v=2
```

Veremos que estamos ejecutando una solicitud a un nuevo pod ya con la version nueva de la app.

9- La actualización se puede confirmar también ejecutando el comando:

```sh
$ kubectl rollout status deployments/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out
```

10- Ahora vamos a verificar que version estan ejecutando los pods:

```sh
$ kubectl describe pods
Name:           kubernetes-bootcamp-7799cbcb86-bkdfg
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:09 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b28af66de45c3738f8d480422cdcee6bdbb04952bb69c2218365f05cb74e3c25
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              12m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-bkdfg to minikube
  Normal  SuccessfulMountVolume  12m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 12m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                12m   kubelet, minikube  Created container
  Normal  Started                12m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-jg2sm
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:09 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://67b1b3be63f43c37a16738aab9f0057546fcc7c4ded0f82e0313cf1d1688e905
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              12m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-jg2sm to minikube
  Normal  SuccessfulMountVolume  12m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 12m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                12m   kubelet, minikube  Created container
  Normal  Started                12m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-mqmcm
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:10 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.11
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://ff45057c81beed58553fe76f106c5ce932e3856dfb1320da621e7fba0c624ef0
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:12 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              12m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-mqmcm to minikube
  Normal  SuccessfulMountVolume  12m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 12m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                12m   kubelet, minikube  Created container
  Normal  Started                12m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-zxcnb
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:10 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://dd2ab77fe99aae43fc48958af209d91728035ed8e5938d69cb15a8b5bb37fa05
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:11 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              12m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-zxcnb to minikube
  Normal  SuccessfulMountVolume  12m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 12m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                12m   kubelet, minikube  Created container
  Normal  Started                12m   kubelet, minikube  Started container
```

Ahora vamos a hacer otra actuaizacion de la imagen y le vamos a poner el tag **v10**

11- Vamos a ejecutar el siguiente comando para desplegar la nueva imagen:

```sh
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
deployment.extensions/kubernetes-bootcamp image updated
```
12- Vamos a ver el status del deployment:

```sh
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         5         2            3           20m
```

**Veremos que hay un detalle, no tenemos el numero deseado de Pods disponibles.**

13- Vamos a verificar los pods que estan corriendo:

```sh
$ kubectl get pods
NAME                                   READY     STATUS             RESTARTS   AGE
kubernetes-bootcamp-5f76cd7b94-j9klm   0/1       ImagePullBackOff   0          2m
kubernetes-bootcamp-5f76cd7b94-rb2tn   0/1       ImagePullBackOff   0          2m
kubernetes-bootcamp-7799cbcb86-bkdfg   1/1       Running            0          17m
kubernetes-bootcamp-7799cbcb86-jg2sm   1/1       Running            0          17m
kubernetes-bootcamp-7799cbcb86-zxcnb   1/1       Running            0          17m
```

14- Si ejecutamos el comando `describe` vamos a obtener mas detalle:

```sh
$ kubectl describe pods
Name:           kubernetes-bootcamp-5f76cd7b94-j9klm
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:47:53 +0000
Labels:         pod-template-hash=1932783650
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Pending
IP:             172.18.0.3
Controlled By:  ReplicaSet/kubernetes-bootcamp-5f76cd7b94
Containers:
  kubernetes-bootcamp:
    Container ID:
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v10
    Image ID:
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          False
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Normal   Scheduled              4m               default-scheduler  Successfully assigned kubernetes-bootcamp-5f76cd7b94-j9klm to minikube
  Normal   SuccessfulMountVolume  4m               kubelet, minikube  MountVolume.SetUpsucceeded for volume "default-token-s62f4"
  Normal   Pulling                2m (x4 over 4m)  kubelet, minikube  pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed                 2m (x4 over 4m)  kubelet, minikube  Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc = unauthorized: authentication required
  Warning  Failed                 2m (x4 over 4m)  kubelet, minikube  Error: ErrImagePull
  Normal   BackOff                2m (x6 over 4m)  kubelet, minikube  Back-off pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed                 2m (x6 over 4m)  kubelet, minikube  Error: ImagePullBackOff


Name:           kubernetes-bootcamp-5f76cd7b94-rb2tn
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:47:53 +0000
Labels:         pod-template-hash=1932783650
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Pending
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5f76cd7b94
Containers:
  kubernetes-bootcamp:
    Container ID:
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v10
    Image ID:
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          False
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Normal   Scheduled              4m               default-scheduler  Successfully assigned kubernetes-bootcamp-5f76cd7b94-rb2tn to minikube
  Normal   SuccessfulMountVolume  4m               kubelet, minikube  MountVolume.SetUpsucceeded for volume "default-token-s62f4"
  Normal   Pulling                2m (x4 over 4m)  kubelet, minikube  pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed                 2m (x4 over 4m)  kubelet, minikube  Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc = unauthorized: authentication required
  Warning  Failed                 2m (x4 over 4m)  kubelet, minikube  Error: ErrImagePull
  Normal   BackOff                2m (x6 over 4m)  kubelet, minikube  Back-off pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed                 2m (x6 over 4m)  kubelet, minikube  Error: ImagePullBackOff


Name:           kubernetes-bootcamp-7799cbcb86-bkdfg
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:09 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b28af66de45c3738f8d480422cdcee6bdbb04952bb69c2218365f05cb74e3c25
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              18m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-bkdfg to minikube
  Normal  SuccessfulMountVolume  18m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 18m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                18m   kubelet, minikube  Created container
  Normal  Started                18m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-jg2sm
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:09 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://67b1b3be63f43c37a16738aab9f0057546fcc7c4ded0f82e0313cf1d1688e905
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              18m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-jg2sm to minikube
  Normal  SuccessfulMountVolume  18m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 18m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                18m   kubelet, minikube  Created container
  Normal  Started                18m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-zxcnb
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:10 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://dd2ab77fe99aae43fc48958af209d91728035ed8e5938d69cb15a8b5bb37fa05
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:11 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              18m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-zxcnb to minikube
  Normal  SuccessfulMountVolume  18m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 18m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                18m   kubelet, minikube  Created container
  Normal  Started                18m   kubelet, minikube  Started container
```

Veremos que no hay una imagen llamada **v10** en el repositorio. Entonces vamos a hacer un rool back a una la version anetrior estable. 

15- Vamos a ejecutar `rollout` para volver a una version estable:

```sh
$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp
```

El comando revierte el deployment hacia una version anterior (v2 de la imagen). Todas las actualizaciones son versionadas y siempre podremos revertir a una version anetrior estable.

16- Vamos a comprobar el estado de los Pods:

```sh
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-7799cbcb86-bkdfg   1/1       Running   0          26m
kubernetes-bootcamp-7799cbcb86-jg2sm   1/1       Running   0          26m
kubernetes-bootcamp-7799cbcb86-n6mpt   1/1       Running   0          2m
kubernetes-bootcamp-7799cbcb86-zxcnb   1/1       Running   0          26m
```

Ahora tenemos los 4 Pods corriendo y a continuacion vamos a verificar que imagen esta desplegada.

17- Vamos a verificar que imagen estan usando los Pods:

```sh
$ kubectl describe pods
Name:           kubernetes-bootcamp-7799cbcb86-bkdfg
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:09 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.9
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b28af66de45c3738f8d480422cdcee6bdbb04952bb69c2218365f05cb74e3c25
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              27m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-bkdfg to minikube
  Normal  SuccessfulMountVolume  27m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 27m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                27m   kubelet, minikube  Created container
  Normal  Started                27m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-jg2sm
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:09 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://67b1b3be63f43c37a16738aab9f0057546fcc7c4ded0f82e0313cf1d1688e905
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              27m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-jg2sm to minikube
  Normal  SuccessfulMountVolume  27m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 27m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                27m   kubelet, minikube  Created container
  Normal  Started                27m   kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-n6mpt
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:57:35 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.4
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://1d1767c6e109f26a154522c27727fd560f495aa7454ffe4af0d92f2fe8400303
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:57:35 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              3m    default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-n6mpt to minikube
  Normal  SuccessfulMountVolume  3m    kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 3m    kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                3m    kubelet, minikube  Created container
  Normal  Started                3m    kubelet, minikube  Started container


Name:           kubernetes-bootcamp-7799cbcb86-zxcnb
Namespace:      default
Node:           minikube/172.17.0.11
Start Time:     Thu, 03 Jan 2019 17:33:10 +0000
Labels:         pod-template-hash=3355767642
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-7799cbcb86
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://dd2ab77fe99aae43fc48958af209d91728035ed8e5938d69cb15a8b5bb37fa05
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:fb1a3ced00cecfc1f83f18ab5cd14199e30adc1b49aa4244f5d65ad3f5feb2a5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 03 Jan 2019 17:33:11 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s62f4 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s62f4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s62f4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              27m   default-scheduler  Successfully assigned kubernetes-bootcamp-7799cbcb86-zxcnb to minikube
  Normal  SuccessfulMountVolume  27m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s62f4"
  Normal  Pulled                 27m   kubelet, minikube  Container image "jocatalin/kubernetes-bootcamp:v2" already present on machine
  Normal  Created                27m   kubelet, minikube  Created container
  Normal  Started                27m   kubelet, minikube  Started container
```

## Final ##

Este articulo pretende ser un resumen del curso oficial de Kubernetes, espero que les sea de utilidad.

### Links de interés: ###

[Kubernetes Documentacion Oficial][Kubernetes Documentacion Oficial]

[Kubernetes Documentacion Oficial]: https://kubernetes.io/docs/tutorials/kubernetes-basics/

