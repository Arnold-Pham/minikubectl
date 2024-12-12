# Comment utiliser Kubernetes en local en tant qu'étudiant fauché

## Préparation de l'environnement

Pour exécuter Kubernetes en local, voici les étapes à suivre:

### Installer les outils nécessaires sur une VM

1. **Docker**: [Guide d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

2. **Docker Compose**: [Guide d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)

3. **Minikube**: [Guide d'installation](https://minikube.sigs.k8s.io/docs/start)

4. **Kubectl**: [Guide d'installation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

### Démarrer un cluster Kubernetes

```sh
# Démarrer un cluster avec un seul noeud
minikube start

# Créer un cluster avec plusieurs noeuds
minikube start -n 2

# Créer un cluster multi-noeuds avec des ressources CPU et RAM définies
minikube start --cpus 4 --memory 4096
```

**Concepts clés**:
- Un **cluster** est un réseau de machines travaillant ensemble pour répondre aux requêtes clients
- Un **noeud** est une machine dans ce réseau
  - Le premier noeud agit comme orchestrateur, distribuant les charges de travail sur les autres noeuds (appelés "workers")

### Vérifications de base

```sh
# Vérifier l'état du cluster
minikube status

# Lister les noeuds du cluster
kubectl get nodes
```

---

## Création des Pods, Deployments et Services

### Méthode 1: Utiliser un fichier de configuration YAML

Créez un fichier `matrix.yaml` avec le contenu suivant:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: matrix-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: matrix-label
  template:
    metadata:
      labels:
        app: matrix-label
    spec:
      containers:
      - name: matrix
        image: fredericeducentre/matrix
---
apiVersion: v1
kind: Service
metadata:
  name: matrix-service
spec:
  selector:
    app: matrix-label
  type: NodePort
  ports:
    - port: 80
```

**Explications**:
- **Deployment**: Définit deux réplicas (instances) de l'image `fredericeducentre/matrix`
- **Service**: Expose les Pods via le port (8080)

Pour appliquer ce fichier:

```sh
kubectl apply -f matrix.yaml
```

### Méthode 2: Utiliser des commandes Kubectl

- Créer un Deployment directement depuis la ligne de commande:

```sh
kubectl create deployment matrix --replicas=2 --image=fredericeducentre/matrix
```

- Exposer ce Deployment en tant que Service:

```sh
kubectl expose deployment matrix --type=NodePort --port=80
```

---

## Vérifications et gestion

### État des ressources

- Vérifier l'état des Pods, Deployments et Services:

```sh
kubectl get pods
kubectl get deployments
kubectl get services
```

- Pour plus de détails sur les Pods:

```sh
kubectl get pods -o wide
```

### Accéder au Service

Une fois le Service déployé, ouvrez-le avec Minikube:

```sh
minikube service matrix-service
```

Cela devrait ouvrir un navigateur affichant les informations du serveur (provenant de l'image `echo-server`)

### Debugging

- Si des problèmes surviennent, consultez les logs des Pods:

```sh
kubectl logs <nom-du-pod>

# Pour surveiller les logs en temps réel:
kubectl logs -f <nom-du-pod>
```

Surveiller les logs permet d’identifier rapidement les erreurs pendant l'exécution

---

## Notes supplémentaires

Supprimez les ressources pour libérer de l'espace:

```sh
# Dans le cas d'un fichier
yamlkubectl delete -f matrix.yaml

# En général
kubectl delete service <nom-du-service>
kubectl delete deployment <nom-du-deployment>

minikube stop
minikube delete
```

---

## Déployer une application avec une base de données

Ce guide explique comment configurer et déployer une application WordPress connectée à une base de données MySQL sur Kubernetes

### Étape 1: Configuration des fichiers de déploiement

Nous utiliserons deux fichiers YAML: un pour WordPress et un pour MySQL

#### Fichier de déploiement WordPress: `wordpress.yaml`

Ce fichier configure WordPress et le Service associé pour exposer l'application

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:6.2.1-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          value: root
        - name: WORDPRESS_DB_USER
          value: wordpress
```

#### Fichier de déploiement MySQL: `mysql.yaml`

Ce fichier configure la base de données MySQL et le Service associé pour permettre la communication avec WordPress

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          value: root
```

### Étape 2: Déployer les ressources

Appliquez les fichiers de configuration avec les commandes suivantes:

```sh
# Déployer la base de données MySQL
kubectl apply -f mysql.yaml

# Déployer WordPress
kubectl apply -f wordpress.yaml
```

### Étape 3: Vérifications

- Vérifiez que les Pods sont créés et en état de fonctionnement:

```sh
kubectl get pods
```

- Vérifiez les Services:

```sh
kubectl get services
```

### Étape 4: Accéder à l'application

Utilisez la commande suivante pour obtenir l'URL de WordPress exposée par Minikube:

```sh
minikube service wordpress
```

Cela ouvrira l'application WordPress dans un navigateur

### Étape 5: Débogage

- En cas de problème, vérifiez les logs des Pods:

```sh
# Logs pour WordPress
kubectl logs -f <nom-du-pod-wordpress>

# Logs pour MySQL
kubectl logs -f <nom-du-pod-mysql>
```
