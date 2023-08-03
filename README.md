
# projet file rouge : partie 1" ( déploiement du site web ODOO par orchestration en K8S)

dans le cadre de la formation de EAZYTRAINING , il nous a été demandé de mettre en place le site web vitrine que nous avons créé dans la partie 1 à l'aide de dockerfile , ce site nous permettre de nous réorienter vers deux site web ODOO PG_ADMIN, la mise en place de ces 3 site web doit respecter une architecture proposée dans l'énoncé du projet que vous trouverez [ici](https://github.com/sadofrazer/ic-webapp "ici")



nous commençons par les étapes qui nous permettra de déployer notre architecture 

## architecture du projet 

Les applications ou services seront déployées dans un cluster Minikube, donc à un seul nœud et devront respecter l’architecture suivante.

<<p align="center">
  <img src="https://github.com/sadofrazer/ic-webapp/blob/master/images/synoptique_Kubernetes.jpeg">
</p>
------------

par la suite les manifestes doivent être crées comme suit : 


## 1- création de NAMESPACE

la création du Namespace se fait par la création d'un manifeste de type YAML, cela permettra de créer un espace de travail spécifique pour ce projet 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: icgroup
  labels:
    name: prod
```
## 2- création de la partie BACKEND

notre application ODOO nécessite le déploiement d'une base de données POSTGRES , cela est nécessaire pour le stockage des données entrée par le site de ODOO .
le déploiement du BD nécessite de déployer les éléments suivants sous forme de manifestes YAML: 
- service ClusterIP : cela rendre la DB disponible aux applications qui se trouve dans le même namespace .
- le secret : un manifeste qui permettre de sécuriser le mot de passe de notre DB
- le manifeste de configuration : permettre de définir les variables d'environnement qui permettre la connexion avec la BD
- le manifeste de déploiement : permettre de lancer notre application avec les paramètres que nous avons défini ( image, port , nombre de réplicas…ETC) 

tous ces manifestes sont classés dans un dossier POSTGRES , pour lancer la DB , il faut se rendre dans le dossier postgres sur le terminal et exécuter la commande suivante : 

```
kubectl apply -f .
```
## 3- création de la partie FRONTEND
### 1- déploiement de site web vitrine ic-webapp:

le site web vitrine nécessite que là l'image que nous avons Builder dans la partie 1' est stocké dans la machine local , ou disponible sur docker hub , cela permettre à K8S de récupérer cette image et déployer un pod a la base de cette dernière , le déploiement de ce site nécessite les éléments suivants : 
- le service Node PORT: ce service rendre notre application disponible à l'extérieur sur le port que nous avons défini sur cette partie
- le manifeste de déploiement : permettre de lancer notre application avec les paramètres que nous avons défini ( image, port , nombre de réplicas…ETC) , et les deux variables qui nous permettre d'accéder à nos deux site web : ODOO_URL et PGADMIN_URL avec les bon adresse IP et port
  
tous ces manifestes sont classés dans un dossier ic-webapp , pour lancer notre site vitrine , il faut se rendre dans le dossier ic-webapp sur le terminal et exécuter la commande suivante : 

```
kubectl apply -f .
```
   
### 2- deployment de ODOO:

le déploiement de ODOO se fait par la mise en place des éléments suivant :
- le service Node PORT: pour rendre notre application disponible sur le port que nous avons déclaré dans cette partie 
- le PV et PVC : PV stockage dans le cluster qui a été provisionné par un administrateur , PVC est une demande de stockage par un utilisateur pour consommer les ressources PV
- le secret : un manifeste qui permettre de sécuriser le mot de passe pour accéder au DB
- le manifeste de déploiement : permettre de lancer notre application avec les paramètres que nous avons défini ( image, port , nombre de réplicas…ETC) et déclarer aussi les bonnes valeurs pour connecter à la DB 
- permission aux répertoires : il faut créer attribuer les droits à tous les répertoires que nous avons déclaré dans nos fichier YAML avant d'exécuter ( dans les hostpath ) avec la commande suivante :
```
sudo mkdir -p /data/postgre-volume /data_docker/addons /data_docker/config
sudo chmod -R 777 /data/postgre-volume /data_docker/addons /data_docker/config
```
tous ces manifestes sont classés dans un dossier ODOO , pour lancer notre site ODOO , il faut se rendre dans le dossier ODOO sur le terminal et exécuter la commande suivante : 

```
kubectl apply -f .
```  

### 2- deployment de ODOO:

le déploiement de PGADMIN, c'est un site web qui permettre de consulter et manager la DB que nous avons rempli à l'aide de ODOO , le déploiement se fait par la mise en place des éléments suivants :

- le service Node PORT: pour rendre notre application disponible sur le port que nous avons déclaré dans cette partie 
- le PV et PVC : PV stockage dans le cluster qui a été provisionné par un administrateur , PVC est une demande de stockage par un utilisateur pour consommer les ressources PV
- le secret : un manifeste qui permettre de sécuriser le mot de passe pour accéder au DB
- le manifeste de déploiement : permettre de lancer notre application avec les paramètres que nous avons défini ( image, port , nombre de réplicas…ETC) et déclarer aussi les bonnes valeurs pour connecter à la DB 
- permission aux répertoires : il faut créer attribuer les droits à tous les répertoires que nous avons déclaré dans nos fichiers YAML avant d'exécuter ( dans les hostpath ) avec la commande suivante :
```
sudo mkdir -p /data_k8s/pgadmin4
sudo chmod -R 777 /data_k8s/pgadmin4
```
tous ces manifestes sont classés dans un dossier pg_admin , pour lancer notre site pg_admin , il faut se rendre dans le dossier pg_admin sur le terminal et exécuter la commande suivante : 
```
kubectl apply -f .
```  

le résultat doit être comme suit :


![image](https://github.com/adda213/projet-file-rouge-partie-2/assets/123883398/a9478483-3b29-47b5-bda8-b0322d5f3c15)


![image](https://github.com/adda213/projet-file-rouge-partie-2/assets/123883398/3dacefd6-76eb-423f-8e54-75b7e1c5f06e)

FIN  

nom : KADDOUR BRAHIM  
Prénom : Adda Zouaoui  
Formation : Eazytrainning BOOTCAMP-12  
