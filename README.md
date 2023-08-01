
# WORDPRESS WITH K8S PROJECT

Dans ce repository se trouve un ensemble de manifestes qui permettra de déployer des applications type frontend (WordPress) et backend (MySQL) et mettre en œuvre les services qui établirons la connexion entre les deux, le déploiement se fait à l'aide de Kubernetes pour orchestrer nos applications (scalabilité, nombre de réplicas ... etc.) 

<p align="center">
  <img src="https://github.com/adda213/mini-projet-K8S/assets/123883398/4ce7c815-98de-45a9-bc6a-54aa2a6e6a7e">
</p>

## architecture du projet 

Les applications ou services seront déployées dans un cluster Minikube, donc à un seul nœud et devront respecter l’architecture suivante.

<p align="center">
  <img src="https://github.com/adda213/mini-projet-K8S/assets/123883398/2fd5d391-9fb0-4223-8ffb-ed1c5ec427ca">
</p>
------------

par la suite les manifestes doivent être crées comme suit : 


## 1- création de NAMESPACE

la création du Name space se fait par la création d'un manifeste de type YAML, cela permettra de créer un espace de travail spécifique pour ce projet 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: wordpress
  labels:
    name: wordpress
```
## 2- création de PVC (Persistant Volume Claim) pour le Backend et Frontend 

le PVC est une demande de stockage par un utilisateur, cela est essentiel pour les deux déploiements pour permettre le stockage et la lecture des donnés (dans ce cas localement).
PS : le PV est une pièce de stockage dans le cluster provisionné par l’administrateur, cette section est décalée dans le même manifeste de déploiement.


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
  namespace : wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
  namespace : wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```
## 3- création des déploiement MySQL et WORDPRESS 

dans cette étape du projet, 2 déploiements doivent être créer pour chacune des application (frontend et backend), à savoir et à ne pas oublier : 
  - MySQL : 
      * il faut déclarer les variables d'environnement qui permettre à WordPress de se connecter à la base de données (`MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_RANDOM_ROOT_PASSWORD`)
      * déclare le service de type CLUSTERIP, pour rendre le backend visible pour les autres applications qui se trouve dans le même cluster
      * déclarer le fichier secret qui contient les mots de passe de la base de données 
```yaml deployment 
# MySQL + CLUSTERIP + PV
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
  namespace : wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
       containers:
       - image: wordpress:latest
         name: wordpress
         env:
         - name: WORDPRESS_DB_HOST
           value: mysql-dep
         - name: WORDPRESS_DB_USER
           value: wordpress
         - name: WORDPRESS_DB_NAME
           value: wordpress
         - name: WORDPRESS_DB_PASSWORD
           valueFrom:
             secretKeyRef:
               name: mysql-secret
               key: wordpress_db_password
         ports:
         - containerPort: 80
           name: wordpress
         volumeMounts:
         - name: wordpress-persistent-storage
           mountPath: /var/www/html
       volumes:
       - name: wordpress-persistent-storage
         persistentVolumeClaim:
           claimName: wp-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: service-wordpress
  labels: 
    app: wordpress
  namespace : wordpress
spec:
  type: NodePort
  selector:
    app: wordpress
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
```   
```yaml

# fichier SECRET  
apiVersion: v1
data:
  login: YWRkYQ==
  mysql_password: YWRkYQ==
  wordpress_db_password: YWRkYQ==
  mysql_random_root_password: YWRkYQ==
kind: Secret
metadata:
  name: mysql-secret
  namespace: wordpress
type: Opaque
```

  - WORDPRESS : 
      * il faut declarer les variables d'environnement qui permettre à WordPress de se connecter à la base de données (`WORDPRESS_DB_HOST`, `WORDPRESS_DB_USER`, `WORDPRESS_DB_NAME`, `WORDPRESS_DB_PASSWORD`)
      * la valeur de `WORDPRESS_DB_HOST` doit être identique au nom de service de CLUSTERIP pour le Frontend (dans ce cas `mysql-dep`)
      * déclarer le service de type Node Port, pour rendre le frontend accessible depuis l'extérieur
      * declarer les secrets dans le même ficher secret crée pour le backend

```yaml
# WORDPRESS + NODEPORT + PV
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
  namespace : wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
       containers:
       - image: wordpress:latest
         name: wordpress
         env:
         - name: WORDPRESS_DB_HOST
           value: mysql-dep
         - name: WORDPRESS_DB_USER
           value: wordpress
         - name: WORDPRESS_DB_NAME
           value: wordpress
         - name: WORDPRESS_DB_PASSWORD
           valueFrom:
             secretKeyRef:
               name: mysql-secret
               key: wordpress_db_password
         ports:
         - containerPort: 80
           name: wordpress
         volumeMounts:
         - name: wordpress-persistent-storage
           mountPath: /var/www/html
       volumes:
       - name: wordpress-persistent-storage
         persistentVolumeClaim:
           claimName: wp-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: service-wordpress
  labels: 
    app: wordpress
  namespace : wordpress
spec:
  type: NodePort
  selector:
    app: wordpress
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
```
## 4- DEPLOIEMENT DES MANIFESTS 

après la création de tous les manifestes, il ne reste qu'à appliquer tous les fichiers, soit par utiliser un fichier de kustumisation.yml, ou bien comme dans ce cas à appliquer la commande suivante dans le répertoire des manifestes : 

```
kubebctl apply -f ./
```

le résultat doit être comme suit : 

![image](https://github.com/adda213/mini-projet-K8S/assets/123883398/291f7651-10d4-44dd-ac65-3e5978459b71)
