# Kubernetes con Minikube – Ejemplo de Servicios

Este proyecto muestra cómo levantar un clúster local con **Minikube** y desplegar una aplicación de prueba (**nginx**) exponiéndola mediante los cuatro tipos principales de servicios de Kubernetes:  
- **ClusterIP**  
- **NodePort**  
- **LoadBalancer**  
- **Ingress**

---

## 📦 Requisitos

- **Ubuntu 24.04** (o equivalente con soporte de virtualización)  
- [Docker](https://docs.docker.com/get-docker/) o [VirtualBox](https://www.virtualbox.org/)  
- [kubectl](https://kubernetes.io/docs/tasks/tools/)  
- [minikube](https://minikube.sigs.k8s.io/docs/start/)  

---

## 🚀 Instalación

### Instalar kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### Instalar minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

---

## 🖥️ Levantar el clúster

```bash
minikube start --driver=docker
kubectl get nodes
kubectl cluster-info
```

Habilitar métricas y dashboard:
```bash
minikube addons enable metrics-server
minikube dashboard
```

---

## 📌 Desplegar aplicación de prueba

Crear un deployment con Nginx:
```bash
kubectl create deployment myapp --image=nginx
kubectl scale deployment myapp --replicas=3
kubectl get deployments
```

---

## 🌐 Tipos de Servicios

### 1. ClusterIP (default, interno)
```bash
kubectl expose deployment myapp   --port=80 --target-port=8080   --type=ClusterIP --name=myapp-clusterip
kubectl get svc myapp-clusterip
```
👉 Accesible solo dentro del clúster.

---

### 2. NodePort (externo vía nodo)
```bash
kubectl expose deployment myapp   --port=80 --target-port=8080   --type=NodePort --name=myapp-nodeport
kubectl get svc myapp-nodeport
minikube ip
```
👉 Acceder en navegador: `http://<minikube-ip>:<nodeport>`

---

### 3. LoadBalancer (IP externa simulada)
```bash
kubectl expose deployment myapp   --port=80 --target-port=8080   --type=LoadBalancer --name=myapp-loadbalancer
kubectl get svc myapp-loadbalancer
minikube tunnel
```
👉 Acceder en navegador: `http://<external-ip>:80`

---

### 4. Ingress (reverse proxy con dominio)
Habilitar ingress:
```bash
minikube addons enable ingress
```

Crear archivo `ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-clusterip
            port:
              number: 80
```

Aplicar:
```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```

Añadir en `/etc/hosts`:
```
<minikube-ip>  myapp.local
```

👉 Acceder en navegador: `http://myapp.local/`

---

## ✅ Verificación

- Servicios:
```bash
kubectl get svc
kubectl describe svc myapp-clusterip
```

- Ingress:
```bash
kubectl get ingress
```

- Pods y Endpoints:
```bash
kubectl get pods -o wide
kubectl get endpoints
```

- Métricas:
```bash
kubectl top nodes
kubectl top pods
```

---

## 📊 Estado final

- Clúster Minikube corriendo en `192.168.49.2`  
- Deployment `myapp` con **3 réplicas de Nginx**  
- Servicios funcionando:  
  - **ClusterIP** → interno  
  - **NodePort** → acceso vía `<minikube-ip>:<nodeport>`  
  - **LoadBalancer** → acceso con `minikube tunnel`  
  - **Ingress** → acceso con `http://myapp.local`  

---
