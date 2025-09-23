# Ansible DevOps Automation Collection

Este repositorio contiene una colección de playbooks de Ansible para automatizar el despliegue de infraestructura DevOps, incluyendo aplicaciones dockerizadas, agentes de Azure Pipelines, y clusters de MicroK8s.

## 📋 Funcionalidades

- **🚀 Despliegue de aplicaciones**: Automatiza la instalación de Docker, clonado de repositorios y despliegue con Docker Compose
- **🔧 Configuración de agentes Azure Pipelines**: Instala y configura agentes de build para Azure DevOps
- **☸️ Instalación de MicroK8s**: Despliega clusters de Kubernetes ligeros para desarrollo
- **🐳 Agentes Docker**: Levanta agentes de Azure Pipelines en contenedores Docker
- **⚙️ Configuración del sistema**: Gestiona usuarios, grupos, configuración de zona horaria y hostname

## 📁 Estructura del Proyecto

```
Ansible/
├── deploy_project.yml       # Despliega Stack-DevOps (MongoDB + Node.js + React)
├── install_azp_agent.yml    # Instala agente nativo de Azure Pipelines
├── install_microk8s.yml     # Instala y configura MicroK8s
├── up_agent_docker.yml      # Levanta agente de Azure Pipelines en Docker
├── linux_hostname.yml       # Configura hostname del sistema
├── timezone.yml            # Configura zona horaria
├── inventory.ini           # Inventario de hosts de Ansible
├── tasks/                  # Tareas modulares reutilizables
│   ├── compose.yml         # Ejecuta docker-compose
│   ├── docker.yml          # Instala Docker CE
│   ├── env.yml             # Maneja variables de entorno
│   ├── env_agent_docker.yml # Variables para agente Docker
│   ├── prereqs.yml         # Instala prerequisitos del sistema
│   ├── project.yml         # Clona y configura proyecto
│   └── user.yml           # Gestiona usuarios y grupos
├── templates/
│   └── dotenv.j2          # Template para archivos .env
└── vars/
    ├── env.yml            # Variables de entorno (generado desde template)
    └── env.yml.template   # Template de configuración
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

