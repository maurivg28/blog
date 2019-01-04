---
title:  "Kubernetes - let's do it"
date:   2019-01-04 10:00:00
categories: [Kubernetes]
tags: [Kubernetes, Containers, Docker]
---
En el articulo anterior vimos toda la parte teorica de Kubernetes. Ahora pretendo mostrarles algo mas practico para tener una idea sobre como funciona un cluster de kubernetes.


## Escenario practico ##

En nuestro caso práctico de instalación y configuración de kudeadm tendremos el siguiente escenario que se compone por tres nodos, de los cuales uno de ellos es el master y los otros dos son nodos que sirven al master.
La intraestructura la vamos a montar sobre Azure y seran 3 VM con Ubuntu 16 LTS.
La siguiente imagen muestra como es el escenario.

**Instalacion de Docker**

En cada nodo vamos a instalarle el motor de Docker. En este blog hay un capitulo dedicado a su instalacion.
Les dejo el link de la documentacion oficial para que lo puedan hacer:

[Docker Instalacion][Docker Instalacion]

[Docker Instalacion]: https://docs.docker.com/install/linux/docker-ce/ubuntu/

## Instalacion y Configuracion de Kubeadm ##

Para comenzar tendremos que instalar las aplicaciones básicas para Ubuntu.

Antes vamos a cambiarnos al usuario root:

```sh
sudo su -
```

```sh
apt-get update && apt-get install -y apt-transport-https
```

Descargamos la clave GPG de kubernetes.

```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
```

Añadiremos el repositorio

```sh
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

Actualizamos paquetes para incluir los de kubernetes.

```sh
apt-get update
```

Vamos a realizar la instalación de los paquetes necesarios para kubeadm.

```sh
apt-get install -y kubelet kubeadm kubectl
```

Una vez que se terminaron de instalar los psquetes anteriores, vamos a ejecutar `kubeadm`. 
Esta ejecución solamente se realiza en el equipo que va a ser **master(Master)**.

```sh
kubeadm init
```

Una vez que ejecutamos el comando `kubeadm init` deberiamos ver algo como lo siguiente:

```
[init] Using Kubernetes version: v1.9.2
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
 [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.05.0-ce. Max validated version: 17.03
 [WARNING Hostname]: hostname "master" could not be reached
 [WARNING Hostname]: hostname "master" lookup kubernetes-1 on 192.168.102.2:53: no such host
 [WARNING FileExisting-crictl]: crictl not found in system path
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [kubernetes-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.15]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 49.003529 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node kubernetes-1 as master by adding a label and a taint
[markmaster] Master kubernetes-1 tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 8212ea.b01e65b8129b03eb
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
 https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

kubeadm join --token 8212ea.b01e65b8129b03eb 10.0.0.15:6443 --discovery-token-ca-cert-hash sha256:54e3489945be576a4edbd3d6f268f5f8bcf8e8ece016709b12060df7828ba751
```

Veremos que al final de la ejecucion nos aparece un comando `kubeadm join` el cual debemos guardar para enlazar los nodos al **master**

```sh
kubeadm join --token 8212ea.b01e65b8129b03eb 10.0.0.15:6443 --discovery-token-ca-cert-hash sha256:54e3489945be576a4edbd3d6f268f5f8bcf8e8ece016709b12060df7828ba751
```

## Configuración de Kubeadm (Entorno de usuario) ##

Vamos a configurar el entorno de usuario para ello vamos a realizar lo siguiente:

**Nota:** Tengan en cuenta que lo que vamos a ejecutar se hace solo en el **master**

```sh
mkdir -p $HOME/.kube
```

Vamos a configurar los archivos de configuración

```sh
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Vamos a cambiar los permisos a dicho directorio y con eso ya tendremos configurado el entorno de usuario.

```sh
chown $(id -u):$(id -g) $HOME/.kube/config
```

Vamos a reiniciar la configuracion ejecutando el domando:

```sh
sudo kubeadm reset
```

## Instalacion del POD Network ##

Deberemos instalar un pod network para comunicar el **master** con los demas nodos que en nuestro caso es para permitir la comunicación con los otros dos nodos. Existen diferentes proyectos que proporcionan pod network para kubernetes, algunos de ellos también apoyan la política de red, siendo en nuestro caso en el que vamos a instalar **“Calico”** y se realiza con el siguiente comando **(se ejecuta solo en el master)**.

```sh
kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

## Union de los nodos ##

Para que puedan comunicarse los diferentes nodos con el master y viceversa tendremos que unir los distintos nodos ya que si no, estarían los tres nodos aislado y de esa forma no realizaría su cometido. Para ello vamos a introducir el siguiente comando en los otros dos nodos que se tienen que unir con el master(dicho comando es el que nos generó el **master** cuando se ejecuto el `kudeadm init` ).

Cuando ejecutemos el comando veremos la siguiente salida:

```
root@nodo1:/home/ubuntu# kubeadm join --token 8212ea.b01e65b8129b03eb 10.0.0.15:6443 --discovery-token-ca-cert-hash sha256:54e3489945be576a4edbd3d6f268f5f8bcf8e8ece016709b12060df7828ba751
[preflight] Running pre-flight checks.
 [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.05.0-ce. Max validated version: 17.03
 [WARNING Hostname]: hostname "kubernetes-2" could not be reached
 [WARNING Hostname]: hostname "kubernetes-2" lookup kubernetes-2 on 192.168.102.2:53: no such host
 [WARNING FileExisting-crictl]: crictl not found in system path
[discovery] Trying to connect to API Server "10.0.0.15:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.0.0.15:6443"
[discovery] Requesting info from "https://10.0.0.15:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "10.0.0.15:6443"
[discovery] Successfully established connection with API Server "10.0.0.15:6443"

This node has joined the cluster:
* Certificate signing request was sent to master and a response
 was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

Luego ejecutamos el mismo comando en el nodo restante y veremos la siguente salida:

```
root@nodo2:/home/ubuntu# kubeadm join --token 8212ea.b01e65b8129b03eb 10.0.0.15:6443 --discovery-token-ca-cert-hash sha256:54e3489945be576a4edbd3d6f268f5f8bcf8e8ece016709b12060df7828ba751
[preflight] Running pre-flight checks.
 [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.05.0-ce. Max validated version: 17.03
 [WARNING Hostname]: hostname "kubernetes-3" could not be reached
 [WARNING Hostname]: hostname "kubernetes-3" lookup kubernetes-3 on 192.168.102.2:53: no such host
 [WARNING FileExisting-crictl]: crictl not found in system path
[discovery] Trying to connect to API Server "10.0.0.15:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.0.0.15:6443"
[discovery] Requesting info from "https://10.0.0.15:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "10.0.0.15:6443"
[discovery] Successfully established connection with API Server "10.0.0.15:6443"

This node has joined the cluster:
* Certificate signing request was sent to master and a response
 was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
root@kubernetes-3:/home/ubuntu# kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
The connection to the server localhost:8080 was refused - did you specify the right host or port?

```

Una vez que ejeutamos el comando **join** en ambos nodos, ya tendremos los dos nodos (nodo1 y nodo2) unidos al master.

## Comprobación de la unión de los nodos con el master ##

Vamos a comprobar que tenemos los dos nodos unidos al **master**. Para ello vamos a ir al master y ejecutar el siguiente comando:

```sh
root@master:/home/ubuntu# sudo kubectl get nodes
NAME          STATUS  ROLES  AGE VERSION
kubernetes-1   Ready  master  1h  v1.9.2
kubernetes-2   Ready  <none>  1h  v1.9.2
kubernetes-3   Ready  <none>  1h  v1.9.2
```

## Final ##

Con esto hemos creado un cluster con kubernetes. Espero que les sea de utilidad.

### Links de interés: ###

[Kubernetes Documentacion Oficial][Kubernetes Documentacion Oficial]

[Kubernetes Documentacion Oficial]: https://kubernetes.io/docs/tutorials/kubernetes-basics/

