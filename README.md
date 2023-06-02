# Ansible playbook para configurar workstations de Mikroways

Este repositorio es un playbook de ansible que deja listo un desktop para
trabajar inmediatamente luego de correrlo. Al momento, depende de dos roles:

* [**mikroways.workstation:**](https://galaxy.ansible.com/mikroways/workstation) role público con aplicaciones usadas y configuración
  del shell
* **mikroways.tools:** role privado con un set de herramientas que usamos a
  diario y fueron exclusivamente desarrolladas por Mikroways. Es opcional.

# Herramientas requeridas

* [direnv](https://direnv.net/)
* [python3](https://www.python.org/downloads/)
* [pyenv](https://github.com/pyenv/pyenv#installation)

# Instalar roles y requerimientos

Primero se deben correr los siguientes comandos para instalar Ansible:

```bash
## Instalamos la version de Python a utilizar
pyenv install 3.9.9

## Creamos el entorno de Python con direnv
direnv allow

## Instalamos Ansible en el entorno de Python
pip3 install -r requirements.txt
```

Luego de instalar ansible, se deben instalar los requerimientos del playbook:

```bash
## Si no pertenece a Mikroways ejecutamos el siguiente comando para instalar los roles
ansible-galaxy role install -r ansible/requirements/roles.yml

## Si pertenece a Mikroways ejecutamos el siguiente comando para instalar los roles
ansible-galaxy role install -r ansible/requirements/roles-mw.yml
```

Para actualizar los requerimientos:

```bash
## Actualizamos el repositorio
git pull

## Si no pertenece a Mikroways actualizamos roles con el siguiente comando:
ansible-galaxy role install -r ansible/requirements/roles.yml --force-with-deps

## Si pertenece a Mikroways actualizamos roles con el siguiente comando:
ansible-galaxy role install -r ansible/requirements/roles-mw.yml --force-with-deps
```

# Ejecutar playbook en local

Antes de continuar, es recomendable realizar resguardos de toda configuración del
usuario donde se ejecute el playbook o realizarlo en un usuario nuevo.

Para ejecutar el playbook en caso de que instalemos desde 0 o que realizemos una
actualización debemos ejecutar el siguiente comando:

```bash
ansible-playbook ansible/playbooks/vm-setup.yml -i ansible/inventory/localhost.yml -K
```

Si perteneces a Mikroways, entonces deberías además correr el siguiente playbook

```bash
ansible-playbook ansible/playbooks/vm-setup-mw.yml -i ansible/inventory/localhost.yml -K
```

# Consideraciones

* Si se está utilizando Pop!\_Os se debe agregar además `-e ansible_distribution=Ubuntu`

* Se aconseja probar el playbook con vagrant para verificar si el SO utilizado
 funcionará con el playbook.

* Si ya utilizabas dotfiles, considerá subir tus cambios porque podrías perder
  alguna de tus personalizaciones.

# Funcionamiento

El playbook sigue la siguiente serie de pasos:

1. Instala paquetes en el sistema tales como docker o podman utilizando el gestor
   de paquetes.
1. Instala herramientas especificas en el directorio: `~/.mikroways/bin` con un
   wrapper para descargar binarios.
1. Configura [asdf](https://asdf-vm.com/) y prepara una serie de plugins y
   versiones de productos.
1. Crea las configuraciones propias del entorno dejandolas a disposicion en el
   directorio `~/.mikroways/dotfiles`.
1. (opcional) Si se indico la instalación de las herramientas de Mikroways,
   entonces estas se instalaran en la carpeta `~/.mikroways/tools`.

# Recomendaciones

Una vez instalado tu desktop con este playbook, te recomendamos que agregues en
`$HOME/.envrc` la siguiente configuración:

```
use asdf
```

De esta forma, la performance del uso de asdf se ve mejorada por no usar los
shims sino la resolución del shim correspondiente.

Por otro lado, si trabajando con **`kubectl`** deja de funcionar el
autocomplete, entonces proveemos el alias **`mw-fix-kube-completion`** que
debería actualizar el autocomplete que se suele romper entre diferentes
versiones de kubectl que se manejan con asdf.

# Usar este playbook en un bastion

Si querés usar directamente este playbook en una vm determinada, proponemos usar
el siguiente comando:

```
ansible-playbook ansible/playbooks/vm-setup.yml [-K] \
  -i SOME_USER@10.10.10.10, \
  -e ansible_user=SOME_USER
```

> Dependiendo del usuario remoto, en el ejemplo **SOME_USER**, debe ser
> especificado tanto en la opción `-i` como en `ansible_user`. Además, para usar
> un inventario inline es **fundamental el uso de la coma al final de la IP**.

# ¿Como probar el entorno en Vagrant?

Simplemente correr:

```bash
## Para crear la maquina virtual
vagrant up

## Para ingresar y verificar el entorno
vagrant ssh
```
