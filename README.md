# Ansible DevOps Automation Collection

Este repositorio contiene una colección de playbooks de Ansible para automatizar el despliegue de infraestructura DevOps, incluyendo aplicaciones dockerizadas, agentes de Azure Pipelines, clusters de MicroK8s y automatización completa de máquinas virtuales.

## 📋 Funcionalidades

- **🚀 Despliegue de aplicaciones**: Automatiza la instalación de Docker, clonado de repositorios y despliegue con Docker Compose
- **🔧 Configuración de agentes Azure Pipelines**: Instala y configura agentes de build para Azure DevOps
- **☸️ Instalación de MicroK8s**: Despliega clusters de Kubernetes ligeros para desarrollo
- **🐳 Agentes Docker**: Levanta agentes de Azure Pipelines en contenedores Docker
- **💻 Automatización de VMs**: Crea automáticamente máquinas virtuales con Debian 13.1.0, red puente y configuración desatendida
- **⚙️ Configuración del sistema**: Gestiona usuarios, grupos, configuración de zona horaria y hostname

## 📁 Estructura del Proyecto

```
Ansible/
├── deploy_project.yml          # Despliega Stack-DevOps (MongoDB + Node.js + React)
├── install_azp_agent.yml       # Instala agente nativo de Azure Pipelines
├── install_microk8s.yml        # Instala y configura MicroK8s
├── up_agent_docker.yml         # Levanta agente de Azure Pipelines en Docker
├── linux_hostname.yml          # Configura hostname del sistema
├── timezone.yml               # Configura zona horaria
├── vm_automation_master.yml    # 🆕 Automatización completa de máquinas virtuales
├── inventory.ini              # Inventario de hosts de Ansible
├── tasks/                     # Tareas modulares reutilizables
│   ├── compose.yml            # Ejecuta docker-compose
│   ├── docker.yml             # Instala Docker CE
│   ├── env.yml                # Maneja variables de entorno
│   ├── env_agent_docker.yml   # Variables para agente Docker
│   ├── prereqs.yml            # Instala prerequisitos del sistema
│   ├── project.yml            # Clona y configura proyecto
│   └── user.yml              # Gestiona usuarios y grupos
├── virtual_machines/          # 🆕 Sistema de automatización de VMs
│   ├── create_custom_iso.yml  # Crea ISO personalizada de Debian 13.1.0
│   └── up_virtual_machine.yml # Crea y configura máquinas virtuales
├── task_virtual_machines/     # 🆕 Configuración y tareas de VMs
│   ├── env.yml               # Configuración de VMs y rutas por defecto
│   └── create_vm.yml         # Tareas individuales de creación de VM
├── templates/
│   └── dotenv.j2             # Template para archivos .env
└── vars/
    ├── env.yml               # Variables de entorno (generado desde template)
    └── env.yml.template      # Template de configuración
```

## 🛠️ Prerequisitos

### Sistema Operativo
- Ubuntu/Debian (probado en Ubuntu 20.04+)
- Usuario con privilegios sudo

### Instalación de dependencias
```bash
# Instalar Git
sudo apt install git

# Instalar Ansible
sudo apt install -y ansible

# Clonar este repositorio
git clone https://github.com/JhoannPV/Ansible.git
cd Ansible
```

## ⚙️ Configuración

### 1. Variables de Entorno
Copia el template y configura las variables necesarias:

```bash
cp vars/env.yml.template vars/env.yml
```

### 2. Configuración de VMs (Para automatización de máquinas virtuales)
Las variables de VMs se configuran automáticamente desde `task_virtual_machines/env.yml`:

```bash
# Las rutas por defecto son:
# - Directorio base VMs: /media/jhoann-bohorquez/JABP/VirtualBox VMs
# - Directorio ISOs: /home/jhoann-bohorquez/vm/iso

# Para personalizar las rutas, editar:
nano task_virtual_machines/env.yml
```

Edita `vars/env.yml` con tus credenciales específicas:

```yaml
# Configuración de MongoDB
MONGO_INITDB_USERNAME: "tu_usuario_mongo"
MONGO_INITDB_PASSWORD: "tu_password_seguro"
MONGO_DB_NAME: "nombre_base_datos"
MONGO_URL: "mongodb://user:pass@mongo-db:27017/"

# Configuración JWT (genera con: openssl rand -hex 32)
JWT_SEED: "tu_jwt_seed_de_64_caracteres"
PORT: 3001
VITE_API_URL: "/api"

# Configuración del Agente Azure Pipelines
AZP_URL: "https://dev.azure.com/tu-organizacion"
AZP_TOKEN: "tu_personal_access_token"
AZP_POOL: "nombre_del_pool"
AZP_AGENT_NAME: "nombre_del_agente"

# Configuración opcional del agente
agent_dir: "/home/sysadmin/agent"
agent_work_dir: "_work"
agent_as_service: true
agent_replace: true
agent_user: "sysadmin"
```

### 2. Inventario
El archivo `inventory.ini` está configurado para ejecutar en localhost. Modifica según tus necesidades:

```ini
[controller]
dev-controlplane-01 ansible_host=localhost ansible_user=sysadmin
```

## 🚀 Uso

### Despliegue Completo de Aplicación (Stack DevOps)
Despliega una aplicación completa con MongoDB, API Node.js y frontend React:

```bash
sudo ansible-playbook -i inventory.ini deploy_project.yml
```

**Lo que hace este playbook:**
- Instala prerequisitos del sistema (curl, git, etc.)
- Instala Docker CE y Docker Compose
- Crea usuario `sysadmin` con permisos Docker
- Clona el repositorio [Stack-DevOps](https://github.com/JhoannPV/Stack-DevOps.git)
- Genera archivo `.env` con las variables configuradas
- Levanta los servicios con `docker-compose up -d`

### Instalación de Agente Azure Pipelines (Nativo)
Instala un agente nativo de Azure Pipelines como servicio systemd:

```bash
sudo ansible-playbook -i inventory.ini install_azp_agent.yml
```

### Agente Azure Pipelines en Docker
Levanta un agente de Azure Pipelines en contenedor Docker:

```bash
sudo ansible-playbook -i inventory.ini up_agent_docker.yml
```

### Instalación de MicroK8s
Instala y configura un cluster Kubernetes ligero:

```bash
sudo ansible-playbook -i inventory.ini install_microk8s.yml
```

### Automatización de Máquinas Virtuales
Crea automáticamente máquinas virtuales con Debian 13.1.0 y configuración desatendida:

```bash
# Ejecutar automatización completa de VMs
ansible-playbook vm_automation_master.yml --ask-become-pass
```

**Lo que hace este playbook:**
- 🔧 Configura automáticamente variables de entorno desde `task_virtual_machines/env.yml`
- 💿 Crea ISO personalizada de Debian 13.1.0 con preseed para instalación desatendida
- 🖥️ Crea máquina virtual con VirtualBox (2GB RAM, 20GB disco, red puente)
- 🌐 Detecta automáticamente el adaptador de red para configuración puente
- ⚡ No requiere intervención manual durante el proceso
- 📁 Utiliza rutas por defecto configurables:
  - VMs: `/media/jhoann-bohorquez/JABP/VirtualBox VMs`
  - ISOs: `/home/jhoann-bohorquez/vm/iso`

**Configuración de VMs individuales:**
```bash
# Solo crear ISO personalizada
ansible-playbook virtual_machines/create_custom_iso.yml --ask-become-pass

# Solo crear máquina virtual (requiere ISO existente)
ansible-playbook virtual_machines/up_virtual_machine.yml --ask-become-pass
```

### Configuración del Sistema
```bash
# Configurar hostname
sudo ansible-playbook -i inventory.ini linux_hostname.yml

# Configurar zona horaria
sudo ansible-playbook -i inventory.ini timezone.yml
```

## 📂 Ubicación de Archivos

| Playbook | Directorio de Despliegue | Descripción |
|----------|-------------------------|-------------|
| `deploy_project.yml` | `/home/sysadmin/ansible_project` | Aplicación Stack DevOps |
| `up_agent_docker.yml` | `/home/sysadmin/agent_project` | Agente Docker |
| `install_azp_agent.yml` | `/home/sysadmin/agent` | Agente nativo |
| `vm_automation_master.yml` | Rutas configurables | Máquinas virtuales y ISOs |

### Configuración de Rutas para VMs

Las rutas de las máquinas virtuales se configuran automáticamente en `task_virtual_machines/env.yml`:

```yaml
# Rutas por defecto (personalizables)
vm_base_directory_default: "/media/jhoann-bohorquez/JABP/VirtualBox VMs"
vm_iso_base_default: "/home/jhoann-bohorquez/vm/iso"

# Configuración de red (detección automática)
bridge_adapter: "{{ ansible_default_ipv4.interface }}"
```

## 🔒 Seguridad

- Los archivos `.env` se crean con permisos `0600` (solo lectura/escritura para el propietario)
- Las variables sensibles no se muestran en los logs de Ansible (`no_log: true`)
- Los tokens y passwords se manejan de forma segura durante la ejecución

## 🐛 Solución de Problemas

### Error: "No se encontraron variables"
```bash
# Asegúrate de que existe el archivo de variables
cp vars/env.yml.template vars/env.yml
# Edita el archivo con tus valores reales
nano vars/env.yml
```

### Error de permisos Docker
```bash
# El playbook automáticamente agrega el usuario al grupo docker
# Si hay problemas, reinicia la sesión o ejecuta:
sudo usermod -aG docker sysadmin
newgrp docker
```

### Servicios no se levantan
```bash
# Verifica el estado de Docker
sudo systemctl status docker

# Verifica los logs del proyecto
cd /home/sysadmin/ansible_project
docker-compose logs
```

### Problemas con VirtualBox o VMs
```bash
# Verificar que VirtualBox esté instalado
vboxmanage --version

# Listar VMs existentes
vboxmanage list vms

# Verificar permisos en directorios de VMs
ls -la "/media/jhoann-bohorquez/JABP/VirtualBox VMs"
ls -la "/home/jhoann-bohorquez/vm/iso"

# Si hay problemas de permisos, crear directorios manualmente:
sudo mkdir -p "/media/jhoann-bohorquez/JABP/VirtualBox VMs"
sudo mkdir -p "/home/jhoann-bohorquez/vm/iso"
sudo chown -R $USER:$USER "/home/jhoann-bohorquez/vm"
```

### Problemas de red en VMs
```bash
# Verificar adaptadores de red disponibles
ip link show

# El playbook detecta automáticamente el adaptador por defecto
# Si necesitas especificar uno manualmente, edita:
nano task_virtual_machines/env.yml
# Cambia: bridge_adapter: "nombre_de_tu_adaptador"
```

## 🔄 Requisitos Adicionales para VMs

### Para usar la automatización de máquinas virtuales:

1. **VirtualBox instalado**
   ```bash
   sudo apt update
   sudo apt install virtualbox virtualbox-ext-pack
   ```

2. **Espacio en disco suficiente**
   - Mínimo 5GB para ISO personalizada
   - Mínimo 20GB por VM creada
   - Espacio adicional para snapshots y logs

3. **Permisos de usuario**
   ```bash
   # Agregar usuario al grupo vboxusers
   sudo usermod -aG vboxusers $USER
   # Reiniciar sesión después de este comando
   ```

4. **ISO de Debian 13.1.0**
   - Se descarga automáticamente si no existe
   - URL: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.1.0-amd64-netinst.iso

## 📚 Referencias

- [Documentación de Ansible](https://docs.ansible.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Azure Pipelines Agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/)
- [MicroK8s](https://microk8s.io/docs)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Debian Preseed](https://wiki.debian.org/DebianInstaller/Preseed)

---

**Desarrollado por:** [JhoannPV](https://github.com/JhoannPV)  
