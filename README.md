# Instala Git
__sudo apt install git__

# Instala Ansible
__sudo apt install -y ansible__

# Clona el repositorio con los archivos de automatización
git clone https://github.com/JhoannPV/Ansible.git

# Navega a la carpeta creada
cd Ansible

## Variables de entorno
1) Copia el template y completa los valores:
	- `cp vars/env.yml.template vars/env.yml`
	- Edita `vars/env.yml` con tus credenciales.
2) Ejecuta el playbook:
	- `sudo ansible-playbook -i inventory.ini deploy_project.yaml`
3) El playbook generará `.env` en `/home/sysadmin/ansible_project/.env` con permisos 0600 y sin mostrar secretos en logs.

# Comando de ejecución
__sudo ansible-playbook -i inventory.ini deploy_project.yaml__

# Ubicación
El proyecto resultado de la ejecución del __ansible-playbook__ se almacenará en la ruta:
/home/sysadmin


