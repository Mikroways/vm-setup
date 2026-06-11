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

Si `uv` no está instalado todavía (el propio playbook lo instala via asdf),
instalalo directamente y luego instalá las dependencias:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
```

Opcionalmente, para evitar errores por rate limiting de GitHub durante la
instalación de plugins de asdf, exportá el token antes de correr el playbook
(ver [Token de GitHub](#token-de-github)):

```bash
export GITHUB_API_TOKEN=...
```

### Equipo con asdf y direnv

Si ya corriste este playbook antes, `uv` y `direnv` estarán instalados. Al entrar
al directorio, direnv activa el entorno automáticamente:

```bash
## Opcional: configurar token de GitHub para evitar rate limiting
cp .envrc.private.sample .envrc.private
# editar .envrc.private y completar GITHUB_API_TOKEN

direnv allow    # solo la primera vez o cuando cambie el .envrc
uv sync
```

Esto deja `ansible-playbook` disponible en el PATH sin necesidad de activar el
venv manualmente. Vagrant también funciona directamente.

Luego de instalar ansible, instalar los roles. `roles-mw.yml` ya incluye todo
lo de `roles.yml`, por lo que los miembros de Mikroways solo necesitan correr
un comando:

```bash
## Si no pertenece a Mikroways:
uv run ansible-galaxy role install -r ansible/requirements/roles.yml

## Si pertenece a Mikroways (incluye el comando anterior):
uv run ansible-galaxy role install -r ansible/requirements/roles-mw.yml
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
uv sync

## Si no pertenece a Mikroways:
uv run ansible-galaxy role install -r ansible/requirements/roles.yml --force-with-deps --force

## Si pertenece a Mikroways (incluye el comando anterior):
uv run ansible-galaxy role install -r ansible/requirements/roles-mw.yml --force-with-deps --force
```

## Ejecutar playbook en local

Antes de continuar, es recomendable realizar resguardos de toda configuración del
usuario donde se ejecute el playbook o realizarlo en un usuario nuevo.

Para ejecutar el playbook en caso de que instalemos desde 0 o que realizemos una
actualización debemos ejecutar el siguiente comando:

```bash
## Si no pertenece a Mikroways:
uv run ansible-playbook ansible/playbooks/vm-setup.yml -i ansible/inventory/localhost.yml -K

## Si pertenece a Mikroways (incluye el playbook anterior):
uv run ansible-playbook ansible/playbooks/vm-setup-mw.yml -i ansible/inventory/localhost.yml -K
```

Si ya tenés la workstation configurada y solo querés instalar o actualizar las
herramientas privadas de Mikroways:

```bash
uv run ansible-playbook ansible/playbooks/vm-setup-mw.yml -i ansible/inventory/localhost.yml \
  -e mw_tools_only=true -K
```

## Consideraciones

* Si se está utilizando Pop!\_OS, copiar el sample de host_vars y descomentar la variable:

  ```bash
  cp ansible/inventory/host_vars/localhost.yml.sample ansible/inventory/host_vars/localhost.yml
  # editar y descomentar ansible_distribution: Ubuntu
  ```

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

## Token de GitHub

El token puede ser **Classic** o **Fine-grained**. Para este uso no se necesita
ningún scope ni permiso — solo autenticación.

> **Nota**: La organización Mikroways bloquea fine-grained tokens con lifetime
> mayor a 366 días. Si usás fine-grained, configurá máximo 365 días de expiración.

* **Classic**: [github.com/settings/tokens/new](https://github.com/settings/tokens/new)
  * _Note_: `mw-asdf-rate-limit`
  * _Expiration_: No expiration
  * Sin seleccionar ningún scope

* **Fine-grained**: [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new)
  * _Token name_: `mw-asdf-rate-limit`
  * _Description_: `Token para evitar rate limiting de GitHub al instalar plugins de asdf`
  * _Expiration_: 365 days
  * _Repository access_: Public repositories
  * Sin permisos adicionales

## Usar este playbook en un bastion

Ansible corre desde tu equipo y se conecta al destino vía SSH. El equipo destino
no necesita tener Ansible instalado.

Si querés usar directamente este playbook en una vm determinada, proponemos usar
el siguiente comando:

```bash
uv run ansible-playbook ansible/playbooks/vm-setup.yml \
  -i SOME_USER@10.10.10.10, \
  -e ansible_user=SOME_USER \
  -K
```

> Dependiendo del usuario remoto, en el ejemplo **SOME_USER**, debe ser
> especificado tanto en la opción `-i` como en `ansible_user`. Además, para usar
> un inventario inline es **fundamental el uso de la coma al final de la IP**.
> Si el usuario remoto tiene sudo sin contraseña, omitir `-K`.

## Uso con Execution Environments

Los Execution Environments (EE) permiten correr los playbooks desde un contenedor.
El contenedor incluye Ansible y todos los roles, por lo que no hace falta
instalarlos en el host. `ansible-navigator` se instala como parte de las
dependencias del proyecto (ver [Instalar roles y requerimientos](#instalar-roles-y-requerimientos)).

El archivo `ansible-navigator.yml` en la raíz del repositorio configura los
defaults para todos los comandos `ansible-navigator`:

* **Imagen**: `ghcr.io/mikroways/vm-setup:latest` (imagen pública, se pullea si hay
  nueva versión en el registry)
* **Modo**: `stdout` (output directo, sin TUI interactiva)
* **Pull policy**: `tag` — con el tag `latest`, siempre verifica si hay una imagen
  más nueva en el registry
* **`GITHUB_API_TOKEN`**: si está definido en el entorno (via direnv o export,
  ver [Token de GitHub](#token-de-github)), se pasa automáticamente al contenedor.
  No hace falta agregarlo a ningún comando.
* **`enable-prompts`**: permite que el prompt interactivo de `-K` (contraseña de
  sudo) sea visible y funcional. Sin esto, el prompt queda descartado internamente
  y el comando se cuelga.

### Build

```bash
## Imagen pública (solo mikroways.workstation):
ansible-builder build -c ansible-builder \
  -f ansible-builder/execution-environment.yml \
  -t vm-setup:latest

## Imagen privada Mikroways (incluye mikroways.tools):
## Requiere la clave SSH privada que tiene acceso a GitLab.
## La clave se inyecta como secreto de build y no queda en la imagen final
## (solo existe en el stage intermedio "galaxy", que no se exporta).
ansible-builder build -c ansible-builder \
  -f ansible-builder/execution-environment-mw.yml \
  -t vm-setup-mw:latest \
  --extra-build-cli-args="--secret id=ssh_key,src=$HOME/.ssh/<tu-clave>"
```

### Uso contra hosts remotos

```bash
## Usando la imagen publicada en ghcr.io (default en ansible-navigator.yml):
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --inventory SOME_USER@10.10.10.10, \
  --extra-vars ansible_user=SOME_USER \
  --extra-vars 'ansible_ssh_extra_args="-o IdentitiesOnly=yes"'

## Usando una imagen buildeada localmente:
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --execution-environment-image localhost/vm-setup:latest \
  --pull-policy never \
  --inventory SOME_USER@10.10.10.10, \
  --extra-vars ansible_user=SOME_USER \
  --extra-vars 'ansible_ssh_extra_args="-o IdentitiesOnly=yes"'
```

### Uso con localhost

El inventario `localhost-ee.yml` usa conexión SSH al host real vía `--network=host`.
El SSH agent del host se monta en el contenedor para autenticación (requiere sshd
corriendo en el host):

```bash
## Usando la imagen publicada en ghcr.io (default en ansible-navigator.yml):
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --container-options='--network=host' \
  --container-options="--volume=$SSH_AUTH_SOCK:$SSH_AUTH_SOCK" \
  --inventory ansible/inventory/localhost-ee.yml \
  -- -K

## Usando una imagen buildeada localmente:
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --execution-environment-image localhost/vm-setup:latest \
  --pull-policy never \
  --container-options='--network=host' \
  --container-options="--volume=$SSH_AUTH_SOCK:$SSH_AUTH_SOCK" \
  --inventory ansible/inventory/localhost-ee.yml \
  -- -K
```

> Si el comando falla con "Too many authentication failures", el agente SSH tiene
> más claves cargadas de las que sshd permite intentar. Solución:
> `ssh-add -D && ssh-add ~/.ssh/id_rsa`

## ¿Como probar el entorno en Vagrant?

```bash
## Para crear la maquina virtual
vagrant up

## Para ingresar y verificar el entorno
vagrant ssh
```

> Si no tenés direnv activo, usar `uv run vagrant up` en lugar de `vagrant up`
> a secas — `uv run` activa el venv y deja `ansible-playbook` disponible para
> el provisioner.

Para probar también las herramientas privadas de Mikroways (`vm-setup-mw.yml`), una vez
que la VM esté levantada:

```bash
vagrant provision --provision-with mw
```

### Probar con Execution Environments

```bash
## Levantar la VM sin provision
vagrant up --no-provision

## Imagen pública — desde ghcr.io (default en ansible-navigator.yml):
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --container-options='--network=host' \
  --inventory ansible/inventory/vagrant.yml

## Imagen pública — buildeada localmente (ver Build):
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --execution-environment-image localhost/vm-setup:latest \
  --pull-policy never \
  --container-options='--network=host' \
  --inventory ansible/inventory/vagrant.yml

## Imagen privada MW — buildeada localmente (ver Build):
## SSH_AUTH_SOCK se monta en el contenedor para ForwardAgent hacia la VM,
## necesario para que la VM autentique contra GitLab al clonar repos privados.
ansible-navigator run ansible/playbooks/vm-setup-mw.yml \
  --execution-environment-image localhost/vm-setup-mw:latest \
  --pull-policy never \
  --container-options='--network=host' \
  --container-options="--volume=$SSH_AUTH_SOCK:$SSH_AUTH_SOCK" \
  --pass-environment-variable SSH_AUTH_SOCK \
  --inventory ansible/inventory/vagrant.yml \
  --extra-vars 'ansible_ssh_extra_args="-o IdentitiesOnly=yes -o ForwardAgent=yes"'
```
