# Ansible playbook para configurar workstations de Mikroways

Este repositorio es un playbook de ansible que deja listo un desktop para
trabajar inmediatamente luego de correrlo. Al momento, depende de dos roles:

* [**mikroways.workstation:**](https://galaxy.ansible.com/mikroways/workstation) role público con aplicaciones usadas y configuración
  del shell
* **mikroways.tools:** role privado con un set de herramientas que usamos a
  diario y fueron exclusivamente desarrolladas por Mikroways. Es opcional.

## Instalar roles y requerimientos

Para correr los playbooks se necesita Ansible instalado en un entorno virtual Python,
gestionado con `uv`. El procedimiento varía según el estado del equipo:

### Primera vez en una máquina nueva

Si asdf y direnv no están instalados todavía (el propio playbook los instala),
instalá `uv` directamente y activá el entorno a mano:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

### Equipo con asdf y direnv

Si ya corriste este playbook antes, el propio playbook habrá instalado `uv` via
asdf. El entorno se activa automáticamente al entrar al directorio:

```bash
direnv allow         # crea el venv y activa el entorno; solo la primera vez o cuando cambia el .envrc
uv pip install -r requirements.txt
```

Luego de instalar ansible, instalar los roles. `roles-mw.yml` ya incluye todo
lo de `roles.yml`, por lo que los miembros de Mikroways solo necesitan correr
un comando:

```bash
## Si no pertenece a Mikroways:
ansible-galaxy role install -r ansible/requirements/roles.yml

## Si pertenece a Mikroways (incluye el comando anterior):
ansible-galaxy role install -r ansible/requirements/roles-mw.yml
```

> **Nota:** Ansible ignora el `ansible.cfg` del repositorio si el directorio tiene
> permisos de escritura para todos (world-writable). Para evitar el warning y que
> se cargue la configuración correctamente, correr una vez:
>
> ```bash
> chmod o-w .
> ```

Para actualizar los roles:

```bash
git pull

## Si no pertenece a Mikroways:
ansible-galaxy role install -r ansible/requirements/roles.yml --force-with-deps --force

## Si pertenece a Mikroways (incluye el comando anterior):
ansible-galaxy role install -r ansible/requirements/roles-mw.yml --force-with-deps --force
```

## Ejecutar playbook en local

Antes de continuar, es recomendable realizar resguardos de toda configuración del
usuario donde se ejecute el playbook o realizarlo en un usuario nuevo.

Para ejecutar el playbook en caso de que instalemos desde 0 o que realizemos una
actualización debemos ejecutar el siguiente comando:

```bash
## Si no pertenece a Mikroways:
ansible-playbook ansible/playbooks/vm-setup.yml -i ansible/inventory/localhost.yml -K

## Si pertenece a Mikroways (incluye el playbook anterior):
ansible-playbook ansible/playbooks/vm-setup-mw.yml -i ansible/inventory/localhost.yml -K
```

Si ya tenés la workstation configurada y solo querés instalar o actualizar las
herramientas privadas de Mikroways:

```bash
ansible-playbook ansible/playbooks/vm-setup-mw.yml -i ansible/inventory/localhost.yml -K \
  -e mw_tools_only=true
```

## Consideraciones

* Si se está utilizando Pop!\_Os se debe agregar además `-e ansible_distribution=Ubuntu`

* Se aconseja probar el playbook con vagrant para verificar si el SO utilizado
 funcionará con el playbook.

* Si ya utilizabas dotfiles, considerá subir tus cambios porque podrías perder
  alguna de tus personalizaciones.

## Funcionamiento

El playbook sigue la siguiente serie de pasos:

1. Instala paquetes en el sistema tales como docker o podman utilizando el gestor
   de paquetes.
1. Instala herramientas especificas en el directorio: `~/.mikroways/bin` con un
   wrapper para descargar binarios.
1. Configura [asdf](https://asdf-vm.com/) y prepara una serie de plugins y
   versiones de productos.
1. Crea las configuraciones propias del entorno dejándolas a disposición en el
   directorio `~/.mikroways/dotfiles`.
1. (opcional) Si se indico la instalación de las herramientas de Mikroways,
   entonces estas se instalaran en la carpeta `~/.mikroways/tools`.

## Recomendaciones

Una vez instalado tu desktop con este playbook, te recomendamos que agregues en
`$HOME/.envrc` la siguiente configuración:

```bash
use asdf
```

De esta forma, la performance del uso de asdf se ve mejorada por no usar los
shims sino la resolución del shim correspondiente.

Por otro lado, si trabajando con **`kubectl`** deja de funcionar el
autocomplete, entonces proveemos el alias **`mw-fix-kube-completion`** que
debería actualizar el autocomplete que se suele romper entre diferentes
versiones de kubectl que se manejan con asdf.

## Usar este playbook en un bastion

Ansible corre desde tu equipo y se conecta al destino vía SSH. El equipo destino
no necesita tener Ansible instalado.

Si querés usar directamente este playbook en una vm determinada, proponemos usar
el siguiente comando:

```bash
ansible-playbook ansible/playbooks/vm-setup.yml [-K] \
  -i SOME_USER@10.10.10.10, \
  -e ansible_user=SOME_USER
```

> Dependiendo del usuario remoto, en el ejemplo **SOME_USER**, debe ser
> especificado tanto en la opción `-i` como en `ansible_user`. Además, para usar
> un inventario inline es **fundamental el uso de la coma al final de la IP**.

## ¿Como probar el entorno en Vagrant?

Simplemente correr:

```bash
## Para crear la maquina virtual
vagrant up

## Para ingresar y verificar el entorno
vagrant ssh
```

Para probar también las herramientas privadas de Mikroways (`vm-setup-mw.yml`), una vez
que la VM esté levantada:

```bash
vagrant provision --provision-with mw
```
