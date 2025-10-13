# Despliegue del cluster con MicroK8s, Vault y External Secrets

Este playbook prepara el cluster MicroK8s (control plane) y despliega la app Calendar usando imágenes de Docker Hub. Requisitos:

- Ubuntu/Debian con sudo
- microk8s instalado o permisos para instalar via snap
- Acceso a Internet para descargar los charts y contenedores

## Inventario

Edita `inventory.ini` y define los hosts `controlplane` y `workers` si deseas utilizar más de una máquina. Para un solo nodo, puedes usar localhost.

```
[controller]
dev-controlplane-01 ansible_host=localhost ansible_user=sysadmin
```

## Ejecutar

1. Ajusta el rango de IPs de MetalLB en `deploy_k8s_cluster.yml` variable `metallb_cidr`.
2. Ejecuta el playbook:

```
ansible-playbook -i inventory.ini deploy_k8s_cluster.yml
```

Al finalizar, verás los servicios expuestos. El frontend estará disponible a través del Ingress en `http://calendar.local` y/o mediante un Service `LoadBalancer` en el puerto 8080. El backend tendrá un Service `LoadBalancer` y también está expuesto vía Ingress en la ruta `/api` y corre en el puerto 3001.

## Notas

- Vault corre en modo dev y no es para producción. Usa almacenamiento persistente y TLS en entornos reales.
- External Secrets Operator se instala vía Helm con `microk8s helm`.
- Las imágenes de Docker usadas se asumen como `docker.io/jhoannpv/calendar-frontend`, `docker.io/jhoannpv/calendar-backend`, `docker.io/jhoannpv/calendar-mongodb`. Cambia si es necesario.