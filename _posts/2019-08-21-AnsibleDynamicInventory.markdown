---
title:  "Ansible Dynamic Inventory"
date:   2019-08-21 21:46:00
categories: [Ansible]
tags: [Ansible, Azure, Cloud, Microsoft, DevOps]
---
Si bien Ansible nos ofrece varias formas de administrar los inventarios a los cuales desplegar nuestra configuracion, senti la necesidad de buscar algo que me facilite el despliegue de diferentes configuraciones sin depender de tener que cambiar dichos inventarios.
Ansible ofrece inventarios dinamicos, que tan solo es un script desarrollado en Python, y que nos ofrece diferentes maneras de usarlo para desplegar nuestras configuraciones como codigo. Asimismo estos scripts fieron desarrollados depenediendo del provider (Azure, AWS, VMWARE, etc).

![AnsibleLogo_Logo]({{ site.baseurl }}/images/Ansible_Logo.png)

## Introducción ##

Como les comentaba, la idea es ver como funciona este inventario dinamico que, apartir de la version de Ansible 2.8 ya viene integrado, por lo cual no tendremos la necesidad de descargarlo.
Pretendo que este articulo sea lo mas practico posible, para que se vea como funciona.

## Manos a la obra ##

***Importante!!!:*** Es muy importante que el nombre del resource group, este todo escrito en minusculas, ya que el script de inventario dinamico, no soporta nombres con letras mayusculas.

1- Vamos a comenzar creando un grupo de recursos en Azure, para ello usaremos AzureCli, que se encuentra disponible directamente desde el portal:

```bash
az group create --resource-group testing-rg --location eastus
```

2- Luego vamos a crear dos maquinas virtuales, con sistema operativo Windows:

```bash
az vm create \
  --resource-group testing-rg \
  --name demo-test-vm1 \
  --image "Win2016Datacenter" \
  --admin-username "Demouser" \
  --admin-password "Demouser@123" \
  --location eastus2
```

```bash
az vm create \
  --resource-group testing-rg \
  --name demo-test-vm2 \
  --image "Win2016Datacenter" \
  --admin-username "Demouser" \
  --admin-password "Demouser@123" \
  --location eastus2
```

3- En este caso vamos a generar un tag para las maquinas virtuales, pero dicho script ofrece otras opciones, por ejemplo podremos desplegar configuraciones cuando la maquina fue recien desplegada e iniciada.

Vamos a crear un tag llamado metricbeat:

```bash
az resource tag --tags metricbeat --id /subscriptions/<YourAzureSubscriptionID>/resourceGroups/ansible-inventory-test-rg/providers/Microsoft.Compute/virtualMachines/demo-test-vm1
```

4- Si usamos la ultima version de Ansible, debemos crear un archivo de configuracion. **Es muy importante que el archivo termine en azure_rm.yml**:

Para nuestro caso lo vamos a llamar **inventarioazure_rm.yml**

```bash
plugin: azure_rm
include_vm_resource_groups:
- testing-rg
auth_source: auto
```

5- Una vez que tenemos el archivo de configuracion listo, podremos ejecutar una prueba y ver si nuestros servidores responden, ejecutando el siguiente comando:

```bash
ansible all -m ping -i ./inventarioazure_rm.yml
```

Cuando ejecutemos el comando, podriamos llegar a tener un error como el siguiente, pero que facilmente se soluciona:

```bash
Failed to connect to the host via ssh: Host key verification failed.
```

Para solucionar el error llamado "Host key verification", se debe esitar el archivo de configuracion de Ansible ```/etc/ansible/ansible.cfg``` y luego setear el siguiente valor:

```bash
host_key_checking = False
```

6- Si volvemos a ejecutar el comando del punto 5, veremos el siguiente output:

```bash
demo-test-vm1_0324 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
demo-test-vm2_8971 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7- Ahora que sabemos que todo funciona, vamos a exportar las variables para que queden disponibles en el equipo donde esta corriendo Ansible, para ello ejecutamos el comando export:

```bash
export AZURE_TAGS=metricbeat
```

8- Nuevamente si ejecutamos el comando para verificar que nuestro inventario reconoce las vm, ejecutamos el siguiente comando:

```bash
ansible all -m ping -i ./inventarioazure_rm.yml
```

Aqui hay un detalle no menor, que si se fijan en el resultado del output, solo responde la vm a la cual le creamos el tag:

```bash
demo-test-vm1 | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
```

Una vez que tenemos todo esto funcionando, resta generar un playbook para desplegar la configuracion.

## Configurando Metricbeat ##

La idea sera generar un playbook el cual instale el paquete de Metricbeat en todas las VM con el tag "metricbeat".

1- Vamos a generar un archivo.

```bash
metricbeat.yml
```

2- Copiaremos el siguiente codigo que armamos para la instalacion de metricbeat:

```bash
- name: Instalacion de Metricbeat en Azure virtual machine
  hosts: all
  vars:
    - version: 7.3.0
  tasks:
  - name: Verificar Metricbeat no instalado
    win_service:
      name: metricbeat
    register: mb_service

  - name: Descargar Metricbeat
    win_get_url:
      url: https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-{{ version }}-windows-x86_64.zip
      dest: C:\metricbeat.zip
    when: mb_service.exists == false

  - name: Descomprimir zip de Metricbeat
    win_unzip:
      src: C:\metricbeat.zip
      dest: C:\Program Files\
      delete_archive: yes
    when: mb_service.exists == false

  - name: Renombrar directorio de Metricbeat
    win_command: cmd.exe /c rename metricbeat-{{ version }}-windows-x86_64 Metricbeat
      args:
      chdir: C:\Program Files\
    when: mb_service.exists == false

  - name: Instalar Metricbeat
    win_shell: .\install-service-metricbeat.ps1
      args:
      chdir: C:\Program Files\Metricbeat\
    when: mb_service.exists == false
```

3- Una vez que tenemos el playbook armado, podremos ejeucutar el siguiente comando para desplegar dicha configuracion:

```bash
ansible-playbook  -i ./inventarioazure_rm.yml  metricbeat.yml
```

### Fin ###

Con esto estaremos desplegando configuraciones mediante Ansible dinamicamente sin tener la necesidad de editar el archivo de inventario. Es muy util para integrarlo enseguida de un despliegue automatico de maquina virtual.


### Links de interés: ###

[Azure Ansible Official Page][Ansible]

[Ansible]: http://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html
