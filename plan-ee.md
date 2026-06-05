# Plan: soporte Execution Environments en vm-setup

## Contexto

El PR #6 (de martinrodriguezbj) agrega soporte para Ansible Execution Environments (EE).
Los EE son ideales para **hosts remotos** (bastion machines). Para localhost tienen
limitaciones reales (BECOME/sudo desde un contenedor).

## Caso de uso

1. **Caso principal**: correr los playbooks contra hosts remotos sin instalar ansible en el host local
2. **Caso secundario**: documentar cómo usarlo con localhost y sus limitaciones

## Estructura de archivos

Nuevo directorio `ansible-builder/` con:

**`execution-environment.yml`** — imagen pública (solo mikroways.workstation):
```yaml
version: 3

images:
  base_image:
    name: ghcr.io/ansible/community-ansible-dev-tools:latest

dependencies:
  galaxy: requirements/roles.yml
```

**`execution-environment-mw.yml`** — imagen privada (incluye mikroways.tools):
```yaml
version: 3

images:
  base_image:
    name: ghcr.io/ansible/community-ansible-dev-tools:latest

dependencies:
  galaxy: requirements/roles-mw.yml
```

Diferencias clave con PR #6:
- Usar `ghcr.io/ansible/community-ansible-dev-tools` en lugar de `quay.io/ansible/ansible-runner`
- **No** incluir `requirements.txt` con pip packages — ansible ya está en la imagen base
- Referenciar los `requirements/roles.yml` ya existentes en lugar de duplicarlos

## Comandos a documentar en README

**Build:**
```bash
# Imagen pública
ansible-builder build -c ansible-builder -f ansible-builder/execution-environment.yml -t vm-setup-ee:latest

# Imagen privada (Mikroways)
ansible-builder build -c ansible-builder -f ansible-builder/execution-environment-mw.yml \
  -t vm-setup-ee-mw:latest \
  --ssh default=$HOME/.ssh/id_ed25519
```

**Uso con hosts remotos:**
```bash
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --eei vm-setup-ee:latest \
  -i ansible/inventory/bastion-inventory.yml \
  --mode stdout
```

**Uso con localhost (con limitaciones):**
```bash
ansible-navigator run ansible/playbooks/vm-setup.yml \
  --eei vm-setup-ee:latest \
  -i ansible/inventory/localhost.yml \
  --execution-environment-volume-mounts "$(pwd)/ansible:/ansible:z" \
  --mode stdout \
  -- -K
```
