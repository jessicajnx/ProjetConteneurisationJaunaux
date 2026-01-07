# Projet Conteneurisation Microservices

Projet de conteneurisation avec Docker et Kubernetes comprenant deux microservices communiquant entre eux :
- **php-service** : Service PHP retournant un prénom
- **rentalservice** : Service Spring Boot Java exposant des endpoints REST

## Table des matières
- [Docker standalone](#docker-standalone)
- [Docker Compose](#docker-compose)
- [Kubernetes](#kubernetes)
- [Gateway Ingress NGINX](#gateway-ingress-nginx)

---

## Docker standalone

### Service PHP (retour du prénom)

- Build: `docker build -t firstname-service:latest php-service`
- Run: `docker run -d --rm -p 8081:80 --name firstname-service firstname-service:latest`
- Test: ouvrir http://localhost:8081 dans votre navigateur (réponse: "Jess")

#### Publication Docker Hub
- Tag: `docker tag firstname-service:latest linuxmint75/firstname-service:latest`
- Push: `docker push linuxmint75/firstname-service:latest`
- Lien de l'image Docker: https://hub.docker.com/r/linuxmint75/firstname-service

### Service Java (Spring Boot)

- Build JAR:
  ```powershell
  cd RentalService
  .\gradlew.bat clean build
  ```
- Build image: `docker build -t rentalservice:latest RentalService`
- Run: `docker run -d --rm -p 8080:8080 --name rentalservice rentalservice:latest`
- Test:
  - http://localhost:8080/bonjour → "bonjour"
  - http://localhost:8080/bonjour-php → "bonjour Jess" (appelle le service PHP)

#### Publication Docker Hub
- Tag: `docker tag rentalservice:latest linuxmint75/rentalservice:1.0`
- Push: `docker push linuxmint75/rentalservice:1.0`
- Pull: `docker pull linuxmint75/rentalservice:1.0`

---

## Docker Compose

Déploiement des deux services avec communication inter-conteneurs via réseau Docker interne.

### Démarrage rapide
```powershell
# Arrêter les anciens conteneurs (si besoin)
docker compose down

# Rebuild les images et démarrer
docker compose build --no-cache
docker compose up -d

# Vérifier le statut
docker compose ps
```

### Test des services
- **PHP seul**: http://localhost:8081 → `Jess`
- **Java seul**: http://localhost:8080/bonjour → `bonjour`
- **Communication Java→PHP**: http://localhost:8080/bonjour-php → `bonjour Jess`

### Arrêter les services
```powershell
docker compose down
```

### Architecture
- Fichier: `docker-compose.yml`
- Services:
  - `php-service`: PHP 8.2 Apache (port 8081)
  - `rentalservice`: Spring Boot Java (port 8080)
- Le service Java appelle le service PHP via `http://php-service/` (résolution DNS interne Docker)

### Notes utiles
- Logs en temps réel: `docker compose logs -f`
- Rebuild sans cache: `docker compose build --no-cache`

---

## Kubernetes

Déploiement sur cluster Kubernetes (Minikube ou Docker Desktop Kubernetes).

### Prérequis
- Kubernetes activé (Docker Desktop: Settings > Kubernetes > Enable)
- kubectl installé

### Configuration de l'URL du service PHP

Le fichier `RentalService/src/main/resources/application.properties` contient deux configurations :
```properties
# Docker Compose (commentée par défaut)
# php.service.url=http://php-service/

# Kubernetes (active par défaut)
php.service.url=http://php-service.default.svc.cluster.local/
```

**Important** : Commenter/décommenter selon l'environnement cible avant de rebuilder l'image.

### Déploiement

1. **Build et chargement des images** (Minikube uniquement)
   ```powershell
   docker compose build
   minikube image load firstname-service:latest
   minikube image load rentalservice:latest
   ```
   
   Pour Docker Desktop, les images locales sont automatiquement accessibles.

2. **Appliquer les manifests Kubernetes**
   ```powershell
   kubectl apply -f k8s-manifests.yaml
   ```

3. **Vérifier le déploiement**
   ```powershell
   kubectl get pods
   kubectl get svc
   ```

### Architecture Kubernetes
- **Fichier**: `k8s-manifests.yaml`
- **Deployments**:
  - `php-service-deployment`: 1 réplica du service PHP
  - `rentalservice-deployment`: 1 réplica du service Java
- **Services**:
  - `php-service`: ClusterIP (accès interne uniquement)
  - `rentalservice`: ClusterIP (accès interne uniquement)

Les services sont exposés via Ingress (voir section suivante).

### Nettoyage
```powershell
kubectl delete -f k8s-manifests.yaml
kubectl delete ingress gateway-ingress
```

---

## Gateway Ingress NGINX

Point d'entrée unique pour accéder aux microservices via des règles de routage HTTP.

### Installation de l'Ingress Controller

**Pour Minikube** :
```powershell
minikube addons enable ingress
```

**Pour Docker Desktop Kubernetes** :
```powershell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

Vérifier l'installation :
```powershell
kubectl get pods -n ingress-nginx
```

### Configuration du fichier hosts

Ajouter l'entrée suivante dans `c:\windows\system32\drivers\etc\hosts` (en mode administrateur) :
```
127.0.0.1 myservice.info
```

### Déploiement de l'Ingress

1. **Appliquer la configuration Ingress**
   ```powershell
   kubectl apply -f ingress.yaml
   ```

2. **Démarrer le tunnel Minikube** (obligatoire)
   ```powershell
   minikube tunnel
   ```
   Laisser cette commande active dans une fenêtre PowerShell séparée.

3. **Vérifier l'Ingress**
   ```powershell
   kubectl get ingress
   kubectl get svc -n ingress-nginx
   ```

### Routes disponibles

Une fois le tunnel actif :

- **Service PHP** : http://myservice.info/php
  - Retourne : `Jess`
  
- **Service Java (endpoint simple)** : http://myservice.info/rental/bonjour
  - Retourne : `bonjour`
  
- **Communication microservices** : http://myservice.info/rental/bonjour-php
  - Retourne : `bonjour Jess`
  - Démontre l'appel interne de `rentalservice` vers `php-service` via le DNS Kubernetes

### Architecture Ingress

```
Internet → myservice.info (127.0.0.1 via tunnel)
    ↓
NGINX Ingress Controller (LoadBalancer)
    ├─ /php → php-service:80 (ClusterIP)
    └─ /rental → rentalservice:8080 (ClusterIP)
            └─ appelle php-service.default.svc.cluster.local
```

Le fichier `ingress.yaml` configure :
- **Host** : `myservice.info`
- **Path rewriting** : `/php` et `/rental` sont réécrits pour rediriger vers les services backend
- **ingressClassName** : `nginx`

### Test complet
```powershell
# Test service PHP
curl http://myservice.info/php -UseBasicParsing

# Test service Java
curl http://myservice.info/rental/bonjour -UseBasicParsing

# Test communication inter-services
curl http://myservice.info/rental/bonjour-php -UseBasicParsing

