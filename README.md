# Instala Git
__sudo apt install git__

# Clona el repositorio con los archivos de automatización
git clone https://github.com/JhoannPV/Ansible.git

# Comando de ejecución
__ansible-playbook -i inventory.ini timezone.yaml linux_hostname.yaml deploy_project.yaml__
