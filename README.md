# Comment utiliser Kubernetes en local en tant qu'étudiant fauché

## Préparation de l'environnement

Pour exécuter Kubernetes en local, voici les étapes à suivre :

### Installer les outils nécessaires sur une VM

  1. **Docker** - [Guide d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

  2. **Docker Compose** - [Guide d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)

  3. **Minikube** - [Guide d'installation](https://minikube.sigs.k8s.io/docs/start)

  4. **Kubectl** - [Guide d'installation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

### Démarrer un cluster Kubernetes

```sh
# Démarrer un cluster avec un seul noeud
minikube start

# Créer un cluster avec plusieurs noeuds
minikube start -n 2

# Créer un cluster multi-noeuds avec des ressources CPU et RAM définies
minikube start --cpus 4 --memory 4092
```

**Concepts clés** :
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

### Méthode 1 : Utiliser un fichier de configuration YAML

Créez un fichier `myserver.yaml` avec le contenu suivant :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myserver-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myserver-label
  template:
    metadata:
      labels:
        app: myserver-label
    spec:
      containers:
      - name: myserver
        image: kicbase/echo-server:1.0
---
apiVersion: v1
kind: Service
metadata:
  name: myserver-service
spec:
  selector:
    app: myserver-label
  type: NodePort
  ports:
    - port: 8080
```

- **Explications** :
  - **Deployment** : Définit deux réplicas (instances) de l'image `kicbase/echo-server:1.0`
  - **Service** : Expose les Pods via le port (8080)

- Pour appliquer ce fichier :

```sh
kubectl apply -f myserver.yaml
```

### Méthode 2 : Utiliser des commandes Kubectl

- Créer un Deployment directement depuis la ligne de commande :

```sh
kubectl create deployment myserver --replicas=2 --image=kicbase/echo-server:1.0
```

- Exposer ce Deployment en tant que Service :

```sh
kubectl expose deployment myserver --type=NodePort --port=8080
```

---

## Vérifications et gestion

### État des ressources

- Vérifier l'état des Pods, Deployments et Services :

```sh
kubectl get pods
kubectl get deployments
kubectl get services
```

- Pour plus de détails sur les Pods :

```sh
kubectl get pods -o wide
```

### Accéder au Service

Une fois le Service déployé, ouvrez-le avec Minikube :

```sh
minikube service myserver-service
```

Cela devrait ouvrir un navigateur affichant les informations du serveur (provenant de l'image `echo-server`).

### Debugging

- Si des problèmes surviennent, consultez les logs des Pods :

```sh
kubectl logs <nom-du-pod>

# Pour surveiller les logs en temps réel :
kubectl logs -f <nom-du-pod>
```

Surveiller les logs permet d’identifier rapidement les erreurs pendant l'exécution.

---

## Notes supplémentaires

Supprimez les ressources pour libérer de l'espace :

```sh
# Dans le cas d'un fichier
kubectl delete -f myserver.yaml

# En général
kubectl delete service <nom-du-service>
kubectl delete deployment <nom-du-deployment>

minikube stop
minikube delete
```
