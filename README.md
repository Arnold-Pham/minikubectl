# Comment faire

- Sur une VM, installer docker, docker compose, minikube, kubectl:

    - [Installer Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)
    - [Installer Docker Compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)
    - [Installer Minikube](https://minikube.sigs.k8s.io/docs/start)
    - [Installer Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

---

- Démarrer un cluster:

```sh
# Démarre un cluster avec un seul noeud
minikube start
# On peut aussi créer un cluster avec plusieurs noeuds
minikube start -n 2
# Encore plus loin, on peut créer un cluster de plusieurs noeuds avec des ressources cpu et ram définis
minikube start --cpus 4 --memory 4064
```

---

- On va ensuite créer les pods, deployments et services.

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
kind: Service
apiVersion: v1
metadata:
  name: myserver-service
spec:
  selector:
    app: myserver-label
  type: NodePort
  ports:
    - port: 8080
```

Dans ce fichier, on peut choisir l'image docker qu'on veut utiliser, son port et plein d'autres choses.

---

- Avec cet exemple qu'on va appeler `myserver.yaml` on va utiliser les commandes suivantes

```sh
# On va prendre le fichier de configuration précédent pour le déployer
kubectl apply -f myserver.yaml
# On va vérifier l'état des pods, des déployments et des services pour voir si ils démarrent bien
kubectl get pods
kubectl get deployements
kubectl get services

# Pour aller plus loin, on peut récupérer plus d'informations des pods avec
kubectl get pods -o wide
```

---

- Quand le service a bien démarré, on peut le lancer avec minikube, et ouvrir une porte sur le service

```sh
minikube service myserver-service
```

Si tout se passe bien, on devrait avoir un navigateur qui s'ouvre avec l'image qu'on a choisi

---

- Si on a un problème on peut toujours récupérer les logs de nos pods avec

```sh
kubectl logs <nom-du-pod>
# On peut aussi garder les logs ouverts avec l'option -f
kubectl logs -f <nom-du-pod>
```

Garder les logs ouverts permet de pouvoir voir les logs en temps réél et pouvoir voir les erreurs pendant notre navigation.
