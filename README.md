# Instala Git
__sudo apt install git__

# Clona el repositorio con los archivos de automatización
git clone https://github.com/JhoannPV/Ansible.git

# Navega a la carpeta creada
cd Ansible

# Comando de ejecución
__ansible-playbook -i inventory.ini timezone.yaml linux_hostname.yaml deploy_project.yaml__

# Ubicación
El proyecto resultado de la ejecución del __ansible-palybook__ se almacenará en la ruta:
/home/sysadmin

