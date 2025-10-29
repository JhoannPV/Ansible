# Kubernetes (MicroK8s) – Calendar App con Services ClusterIP

Este documento describe cómo desplegar la aplicación Calendar (MongoDB + Backend + Frontend) en MicroK8s usando Services de tipo ClusterIP, sin LoadBalancer y sin defaults implícitos. El frontend usa Nginx y enruta `/api` hacia el backend (puerto 3001) dentro del cluster.

## Requisitos

- MicroK8s instalado y operativo en el nodo controlador (controller).
- Ansible instalado en tu máquina de automatización.
- Inventario con un host `controller` que tenga MicroK8s y acceso `kubectl` local.
- Variables definidas explícitamente en `Ansible/vars/env.yml` (sin defaults en plantillas).

## Variables necesarias (vars/env.yml)

Debes definir todas estas claves antes de desplegar:

- Imágenes:
	- `FRONTEND_IMAGE`
	- `BACKEND_IMAGE`
	- `MONGODB_IMAGE`
- Namespace y hostname:
	- `K8S_NAMESPACE`
	- `HOST_NAME` (para exponer con Ingress)
- Backend/Secret:
	- `MONGO_INITDB_USERNAME`, `MONGO_INITDB_PASSWORD`, `MONGO_DB_NAME`, `MONGO_URL`
	- `JWT_SEED`, `PORT`, `VITE_API_URL`
<!-- Sin variable APP_NAME: se usa únicamente K8S_NAMESPACE y variables existentes -->

Sugerencia: parte de `Ansible/vars/env.yml.template` y complétalo.

## Orden de despliegue

1) MongoDB (Deployment + Service ClusterIP)
2) Backend (Deployment + Service ClusterIP puerto 3001)
3) Frontend (ConfigMap Nginx + Deployment + Service ClusterIP puerto 80)

## Playbooks

1) Variables de entorno

```bash
cp Ansible/vars/env.yml.template Ansible/vars/env.yml
nano Ansible/vars/env.yml
```

2) Desplegar con Services ClusterIP (Mongo → Backend → Frontend)

```bash
ansible-playbook -i Ansible/inventory.ini Ansible/deploy_clusterip.yml
```

3) Exponer por Ingress (requerido `HOST_NAME` y `K8S_NAMESPACE` definidos)

```bash
ansible-playbook -i Ansible/inventory.ini Ansible/expose_ingress.yml
```

## Acceso sin Ingress (port-forward)

```bash
# Backend en 3001
microk8s kubectl -n <ns> port-forward svc/calendar-backend 3001:3001

# Frontend en 8080
microk8s kubectl -n <ns> port-forward svc/calendar-frontend 8080:80
```

Luego abre http://localhost:8080. El frontend hablará al backend usando `/api` (proxy a 3001).

## Características del frontend (calendar-frontend)

- Réplicas: 3 (puedes escalar dinámicamente con kubectl)
- Readiness: HTTP GET `/` en puerto 80
- Resources: requests `cpu=100m, memory=128Mi`; limits `cpu=250m, memory=256Mi`

## Observabilidad (comandos útiles)

Reemplaza `<ns>` por tu `K8S_NAMESPACE`.

```bash
# Pods del frontend (y nodo donde corren)
microk8s kubectl -n <ns> get pods -l app=calendar-frontend -o wide

# Eventos del namespace
microk8s kubectl -n <ns> get events --sort-by=.lastTimestamp

# Logs de un pod del frontend
POD=$(microk8s kubectl -n <ns> get pods -l app=calendar-frontend -o jsonpath='{.items[0].metadata.name}')
microk8s kubectl -n <ns> logs "$POD"

# Describe del Deployment y el Service del frontend
microk8s kubectl -n <ns> describe deploy/calendar-frontend
microk8s kubectl -n <ns> describe svc/calendar-frontend
```

Registra 1–2 hallazgos para la evidencia (tiempos de readiness, ausencia de warnings, distribución de réplicas por nodo, etc.).

```text
Hallazgos:
- El readiness del frontend fue exitoso en ~N segundos; sin eventos de Warning.
- Las 3 réplicas quedaron programadas en workers diferentes (según disponibilidad).
```

## Operaciones (demo)

```bash
# Escalar el frontend a 5 réplicas
microk8s kubectl -n <ns> scale deploy/calendar-frontend --replicas=5
microk8s kubectl -n <ns> get deploy/calendar-frontend

# Reiniciar el rollout del frontend y ver el estado/historial
microk8s kubectl -n <ns> rollout restart deploy/calendar-frontend
microk8s kubectl -n <ns> rollout status deploy/calendar-frontend --timeout=300s
microk8s kubectl -n <ns> rollout history deploy/calendar-frontend
```

## Notas

- Dentro de MicroK8s no aplica `host.docker.internal`; el backend usa el DNS del Service `mongo-db:27017`.
- Las plantillas no tienen defaults; los playbooks validan y fallan al inicio si faltan variables. Asegúrate de completar `Ansible/vars/env.yml`.
- Los manifests renderizados se guardan en `/tmp/calendar-k8s` en el nodo controller durante el despliegue.

