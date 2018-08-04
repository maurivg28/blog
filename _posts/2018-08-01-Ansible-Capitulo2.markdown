---
title:  "Ansible - Capítulo 2"
date:   2018-08-01 21:46:00
categories: [Ansible]
tags: [Ansible, Azure, Cloud, Microsoft, DevOps]
---
En el capítulo anterior tratamos los conceptos básicos sobre como funciona Ansible. En particular hoy les voy a contar sobre una automatización que tuve que desarrollar para una empresa de medios de pago, la cual tienen un sistema montado sobre servidores web con IIS (backend y frontend), varios sitios, certificados, diferentes url´s, diferentes ambientes (Producción y Staging), y que al momento de ejecutar el playbook sepa utilizar los valores que son destinados para FrontEnd o Backend, etc. Bastante complejo y divertido. Lo interesante de todo esto fue que pudimos aplicar bastantes opciones de las que Ansible nos ofrece.

![AnsibleLogo_Logo]({{ site.baseurl }}/images/Ansible_Logo.png)

## Introducción ##

Ansible es una herramienta open-source desarrollada en python. Actualmente la definen como un motor de orquestación muy simple que automatiza las tareas necesarias. La palabra clave en esta definición es simple y es que Ansible posee unas características que junto su amplia e inteligible documentación hacen esta herramienta realmente atractiva para aquellos que aún no automatizan sus tareas de administración ya que su curva de aprendizaje crece muy rápido.

## Desafio ##

El objectivo era el siguiente:
- Realizar diferentes configuraciones a nivel de sistema operativo, como ser: creación de usuario, cambio de zona horaria, configuración de archivo host.
- Instalacion de Git y clonado del repositorio correspondiente dependiendo si es FrontEnd o BackEnd.
- Desplegar la configuracion correspondiente en el servidor FrontEnd y en el servidor BackEnd. Asimismo desplegar las configuraciones que tienen común ambos servidores. Sumado a que se desplegaran estas configuraciones dependiendo del ambiente Producción o Staging.

## Estructura de la solución ##

```bash
.
├── environments <- environment
|    └── production
|    |    └── group_vars
|    |    |    └── all
|    |    |        └── frontend_production
|    |    |              └── variables.yml
|    |    |        └── backend_production
|    |    |              └── variables.yml
|    |    └── hosts
|    └── staging
|    └── group_vars
|        |    └── all
|        |        └── frontend_production
|        |              └── variables.yml
|        |        └── backend_production
|        |              └── variables.yml
|        └── hosts
├── frontend_prod.yml <-playbook
├── backend_prod.yml <-playbook
├── frontend_staging.yml <-playbook
├── backend_staging.yml <-playbook
└── roles
    └── certificates
        ├── task <- fichero de tareas
        │   └── frontend.yml
        │   └── backend.yml
        │   └── main.yml
        └── default <- archivos con las variables
            └── main.yml
            └── frontend.yml
            └── backend.yml
```


## Diccionarios y loops ##

Para este caso, me estaba enfrentando a un servidor bastante divertido de pensar el "como" automatizarlo con Ansible de la manera mas optima posible, partiendo de la base que contaba con muchos sitios de IIS, los cuales cada uno tenia que correr su aplication pool, su usuario correspondiente, sumandole a todo esto de que cada sitio tenia que tener diferentes bindings, con sus respèctivas url´s y su respectivo certificado.

Una manera que encontré para cumplir con el objectivo fue trabajar con un diccionario y subdiccionarios dentro del mismo, de manera que cuando la tarea sea ejecutada se vayan recorriendo y llamando a las variables correspondientes.

Como se puede apreciar, en el siguiente ejemplo, utilizo para el playbook que se ejecuta dentro del rol el loop "with_dict" para recorrer el diccionario.

```bash
- name: creo los applications pools
  win_iis_webapppool:
    name: "{{ item.value.name }}"
    state: started
    attributes:
      managedRuntimeVersion: v4.0
      managedPipelineMode: Integrated
      processModel.identityType: SpecificUser
      processModel.userName: "{{ application_pool_username }}"
      processModel.password: "{{ application_pool_pass }}"
      processModel.loadUserProfile: True
  with_dict: "{{ iis_front_settings }}"
  tags:
    - FrontEnd

- name: creo los sitios de IIS
  win_iis_website:
    name: "{{ item.value.name }}"
    application_pool: "{{ item.value.application_pool }}"
    physical_path: "{{ item.value.physical_path }}"
    hostname: "{{ item.value.hostname }}"
    state: started
  with_dict: "{{ iis_front_settings }}"
  tags:
    - FrontEnd

- name: agrego los HTTPS binding
  win_iis_webbinding:
    name: "{{ item.value.name }}"
    protocol: https
    port: "{{ item.value.port }}"
    ip: '*'
    host_header: "{{ item.value.host_header }}"
    certificate_hash: "{{ item.value.certificate_hash }}"
    ssl_flags: "{{ item.value.ssl_flags }}"
    state: present
  with_dict:
    - "{{ iis_front_settings }}"
  tags:
    - FrontEnd
```

De esta manera queda solucuionado el problema de cargar varios valores diferenetes en un mismo sitio de iis y asociarle diferentes configuraciones.

## ¿FrontEnd o BackEnd? ##

El segundo desafio con el que me enfrentaba, era que tenia que desplegar configuraciones para servidores diferentes usando las mismas recetas (playbooks).
La solución fue crear dentro de cada rol de ansible, en la carpeta defaults un archivo de variables que tenga una variable vacia, la cual la vamos a llamar **server**.
Luego dentro de cada rol cree tres playbooks, frontend.yml, backend.yml, main.yml. Ahora se los paso a detallar:

- Dentro de frontend.yml y backend.yml agrego las configuraciones particulares para cada servidor.

- Dentro de main.yml, agrego las configuraciones que tendran en común ambos servidores, ya que ansible siempre lee primero los archivos llamados main.yml. El truco esta en que para que ansible sepa que configuracion aplicar, vamos a utilizar las siguientes conficiones:

```bash
- include_tasks: frontend.yml
  when: server == "FrontEnd"

- include_tasks: backend.yml
  when: server == "BackEnd"
```

Hasta acá podemos ejecutar playbooks creados en ansible, los cuales desplegaran diferentes configuraciones, usando diferentes roles y aplicando las mismas tanto para un servidor como para otro, dependiendo del tipo de servidor.
Luego a nivel del playbook, vamos a invocar los diferentes roles llamando a su variable correspondiente (FrontEnd o BackEnd).

```bash
- hosts: front_production
  roles:
    - { role: iis }
    - { role: git, server: 'FrontEnd' }
    - { role: server, server: 'FrontEnd' }
    - { role: certificates, server: 'FrontEnd' }
    - { role: iis_site, server: 'FrontEnd' }
    - { role: metricbeat-windows }
````

En el ejemplo anterior estamos llamando a diferentes roles y ejecutando las diferentes tareas de cada rol para todos los servidores que actuaran como FrontEnd.

Ahora... ¿que sucede cuando nos piden desplegar diferentes ambientes?

### Trabajando con ambientes ###

Ansible nos permite manejar los llamados "environments" para aplicar diferentes configuraciones o las mismas configuraciones pero diferenciando el tipo de servidor.
Cómo podrán apreciar en el siguiente extracto de la estructura, podemos separar los diferentes ambientes. Dicha estructura contiene los archivos de variables con los valores correspondientes a cada ambiente.

```bash

├── environments <- environment
|    └── production
|    |    └── group_vars
|    |    |    └── all
|    |    |        └── frontend_production
|    |    |              └── variables.yml
|    |    |        └── backend_production
|    |    |              └── variables.yml
|    |    └── hosts
|    └── staging
|    └── group_vars
|        |    └── all
|        |        └── frontend_production
|        |              └── variables.yml
|        |        └── backend_production
|        |              └── variables.yml
|        └── hosts
```
### ¿Que pasa con los datos sencibles? ###

Siempre nos preguntamos que pasa con las contraseñas y otro tipo de datos sencibles, sobre lo que no queremos que queden en ninguna receta (playbook), o cualquier archivo de variables.
En mi caso utilice Ansible Vault, el cual me brinda varias opciones. En mi caso decidi por encriptar solo las variables que contenian contraseñas.

### ¿Como se aplica? ###

Y llegamos a la parte final y mas divertida que es cuando ponemos a prueba lo que desarrollamos.
Para aplicar la configuracion sobre los diferentes ambientes que tenemos debemos ejecutar los siguientes comandos de ansible:

```bash
## Produccion

ansible-playbook backend_production.yml --extra-vars "ansible_user=$(AdminUser) ansible_password=$(AdminPassword)" -i environments/production --vault-password-file $(PathPasswordFile)

ansible-playbook frontend_production.yml --extra-vars "ansible_user=$(AdminUser) ansible_password=$(AdminPassword)" -i environments/production --vault-password-file $(PathPasswordFile)

## Staging

ansible-playbook backend_staging.yml --extra-vars "ansible_user=$(AdminUser) ansible_password=$(AdminPassword)" -i environments/staging --vault-password-file $(PathPasswordFile)

ansible-playbook frontend_staging.yml --extra-vars "ansible_user=$(AdminUser) ansible_password=$(AdminPassword)" -i environments/staging --vault-password-file $(PathPasswordFile)
```
Lo que hacen estos comandos es realizar la siguiente ejecucion:

- Aplicar la configuracion del playbook.

- Pasar como variables el usuario y contraseña para la conexion por WINRM a los servidores.

- Ir a buscar los valores de los archivos de variables, dependiendo del ambiente.

- Pedir o tomar de una variable o archivo la contraseña para desencriptar los datos sencibles.

Bueno, a grandes razgos creo que vimos bastantes herramientas que podemos utilizar con ansible, como ser roles, control de ambientes, diccionarios, etc.

Espero que les sea útil.

[Azure Ansible Official Page][Ansible]

[Ansible]: http://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html
