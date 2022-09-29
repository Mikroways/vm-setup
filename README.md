# Ansible playbook para configurar workstations de Mikroways

Este repositorio es un playbook de ansible que deja listo un desktop para
trabajar inmediatamente luego de correrlo. Al momento, depende de dos roles:

* **mikroways.workstation:** role público con aplicaciones usadas y configuración
  del shell
* **mikroways.tools:** role privado con un set de herramientas que usamos a
  diario y fueron exclusivamente desarrolladas por Mikroways. Es opcional.

## Herramientas requeridas

* [direnv](https://direnv.net/)
* [python3](https://www.python.org/downloads/)

## ¿Cómo utilizar?

### Instalar roles y requerimientos

Primero se deben correr los siguientes comandos para instalar Ansible:

```bash
## Creamos el entorno de Python con direnv
direnv allow

## Instalamos Ansible en el entorno de Python
pip3 install -r requirements.txt
```

Luego de instalar ansible, se deben instalar los requerimientos del playbook:

```bash
## Instalamos las colecciones necesarias
ansible-galaxy collection install -r ansible/requirements/collections.yml

## Si no pertenece a Mikroways ejecutamos el siguiente comando para instalar los roles
ansible-galaxy role install -r ansible/requirements/roles.yml

## Si pertenece a Mikroways ejecutamos el siguiente comando para instalar los roles
ansible-galaxy role install -r ansible/requirements/roles-mw.yml
```

Para actualizar los requerimientos:

```bash
## Actualizamos el repositorio
git pull

## Actualizamos las colecciones
ansible-galaxy collection install -r ansible/requirements/collections.yml --force-with-deps

## Si no pertenece a Mikroways actualizamos roles con el siguiente comando:
ansible-galaxy role install -r ansible/requirements/roles.yml --force-with-deps

## Si pertenece a Mikroways actualizamos roles con el siguiente comando:
ansible-galaxy role install -r ansible/requirements/roles-mw.yml --force-with-deps
```

### Ejecutar playbook en local

Antes de continuar, es recomendable realizar resguardos de toda configuración del
usuario donde se ejecute el playbook o realizarlo en un usuario nuevo.

Para ejecutar el playbook en caso de que instalemos desde 0 o que realizemos una
actualización debemos ejecutar el siguiente comando:

```bash
ansible-playbook ansible/playbooks/vm-setup.yml -i ansible/inventory/localhost.yml -K
```

#### Consideraciones

> Si se quieren instalar las herramientas privadas de mikroways se debe agregar
> `-e mw_tools_enabled=true`

> Si se está utilizando Pop!\_Os se debe agregar además
> `-e ansible_distribution=Ubuntu`

> Se aconseja probar el playbook con vagrant para verificar si el SO utilizado
> funcionará con el playbook.

> Si ya utilizabas dotfiles, considerá subir tus cambios porque podrías perder
> alguna de tus personalizaciones.

#### Funcionamiento

El playbook sigue la siguiente serie de pasos:

1. instala paquetes en el sistema tales como docker o podman utilizando el gestor
   de paquetes.
1. instala herramientas especificas en el directorio: `~/.mikroways/bin` con un
   wrapper para descargar binarios.
1. crea las configuraciones propias del entorno dejandolas a disposicion en el
   directorio `~/.mikroways/dotfiles`.
1. (opcional) si se indico la instalacion de las herramientas de Mikroways,
   entonces estas se instalaran en la carpeta `~/.mikroways/tools`.

## ¿Como probar el entorno en Vagrant?

Simplemente correr:

```bash
## Para crear la maquina virtual
vagrant up

## Para ingresar y verificar el entorno
vagrant ssh
```
