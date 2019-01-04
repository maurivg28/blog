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

En caso que tengamos que reiniciar la configuracion ejecutaremos el comando:

```sh
kubeadm reset
```

## Instalacion del POD Network ##

Deberemos instalar un pod network para comunicar el **master** con los demas nodos que en nuestro caso es para permitir la comunicación con los otros dos nodos. Existen diferentes proyectos que proporcionan pod network para kubernetes, algunos de ellos también apoyan la política de red, siendo en nuestro caso en el que vamos a instalar **“Calico”** y se realiza con el siguiente comando **(se ejecuta solo en el master)**.

```sh
kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

## Union de los nodos ##

Para que puedan comunicarse los diferentes nodos con el master y viceversa tendremos que unir los distintos nodos ya que si no, estarían los tres nodos aislado y de esa forma no realizaría su cometido. Para ello vamos a introducir el siguiente comando en los otros dos nodos que se tienen que unir con el master(dicho comando es el que nos generó el **master** cuando se ejecuto el `kudeadm init` ).


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

