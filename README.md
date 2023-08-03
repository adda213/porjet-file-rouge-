
# projet file rouge : partie 1" ( deployement du site web ODOO par orchestation en K8S)

dans le cadre de la formation de EAZYTRAINING , il nous a été demadné de mettre en place le site web vitrine que nous avons crée dans la partie 1 a l'aide de dockerfile , ce site nous premettre de nous reorienter ver deux site web ODOO PG_ADMIN, la mise en place de ces 3 site web doit respecter un architecture proposé dans l'énoncé du projet que vous trouverez [ici](https://github.com/sadofrazer/ic-webapp "ici")



nous commoncons par les étapes qui nous permttra de deployer notre archicteture 

## architecture du projet 

Les applications ou services seront déployées dans un cluster Minikube, donc à un seul nœud et devront respecter l’architecture suivante.

<<p align="center">
  <img src="https://github.com/sadofrazer/ic-webapp/blob/master/images/synoptique_Kubernetes.jpeg">
</p>
------------

par la suite les manifestes doivent être crées comme suit : 


## 1- création de NAMESPACE

la création du Name space se fait par la création d'un manifeste de type YAML, cela permettra de créer un espace de travail spécifique pour ce projet 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: icgroup
  labels:
    name: prod
```
## 2- création de la partie BACKEND

notre application ODOO necessite le deployement d'une base de donnée POSTGRES , cela est necessaire pour le stockage des données entrée par le site de ODOO .
le deployement du BD necessite de deployer les élements suivants sous forme de manifestes YAML: 
- service ClusterIP : cela rendre la DB disponible au applications qui se trouve dans le meme namespace .
- le secret : un manifeste qui permettre de securisé le mot de passe de notre DB
- le manifeste de configuration : permettre de definir les variables d'environement qui permettre la connexion avec la BD
- le manifeste de deployement : permettre de lancer notre application avec les paramettre que nous avons definit ( image, port , nombre de replicas.....ETC) 

tous ces manifestes sont classé dnas un dossier POSTGRES , pour lancer la DB , il faut ce rendre dans le dosier postgres sur le terminal et executer la commande suivante : 

```
kubectl apply -f .
```
## 3- création de la partie FRONTEND
### 1- deployement de site web vitrine ic-webapp:

le site web vitrine necessite que la l'image que nous avons buildé dans la partie 1' est stocké dans la machine local , ou disponible sur dockerhub , cela permettre a K8S de recuprer cette image et deployer un pod a la base de cette derniere , le deployement de ce site necessite les élements suivants : 
- le service nodePORT: ce service rendre notre application disponible a l'exterieur sur le port que nous avons définit sur cette partie
- le manifeste de deployement : permettre de lancer notre application avec les paramettre que nous avons definit ( image, port , nombre de replicas.....ETC) , et les deux variables qui nous permettre d'acceder a nos deux site web : ODOO_URL et PGADMIN_URL avec les bon adresse IP et port
  
tous ces manifestes sont classé dans un dossier ic-webapp , pour lancer notre site vitrine , il faut ce rendre dans le dossier ic-webapp sur le terminal et executer la commande suivante : 

```
kubectl apply -f .
```
   
### 2- deployement de ODOO:

le deployement de ODOO se fait par la mise en place des elements suivant :
- le service nodePORT: pour rendre notre application disponible sur le port que nous avons declaré dans cette partie 
- le PV et PVC : PV stockage dans le cluster qui a été provisionné par un administrateur , PVC est une demande de stockage par un utilisateur pour consommer les ressources PV
- le secret : un manifeste qui permettre de securisé le mot de passe pour acceder au DB
- le manifeste de deployement : permettre de lancer notre application avec les paramettres que nous avons definit ( image, port , nombre de replicas.....ETC) et declarer aussi les bon valeurs pour connecter a la DB 
- permission au repertoires : il faut creer attribuer les droits a tous les repertoires que nous avons declaré dans nos fichier YAML avant d'executer ( dans les hostpath ) avec la commande suivante :
```
sudo mkdir -p /data/postgre-volume /data_docker/addons /data_docker/config
sudo chmod -R 777 /data/postgre-volume /data_docker/addons /data_docker/config
```
tous ces manifestes sont classé dans un dossier ODOO , pour lancer notre site ODOO , il faut ce rendre dans le dossier ODOO sur le terminal et executer la commande suivante : 

```
kubectl apply -f .
```  

### 2- deployement de ODOO:

le deployement de PGADMIN, c'est un site web qui permettre de consulter et manager la DB que nous avons remprli a l'aide de ODOO , le deployement se fait par la mise en place des elements suivants :

- le service nodePORT: pour rendre notre application disponible sur le port que nous avons declaré dans cette partie 
- le PV et PVC : PV stockage dans le cluster qui a été provisionné par un administrateur , PVC est une demande de stockage par un utilisateur pour consommer les ressources PV
- le secret : un manifeste qui permettre de securisé le mot de passe pour acceder au DB
- le manifeste de deployement : permettre de lancer notre application avec les paramettres que nous avons definit ( image, port , nombre de replicas.....ETC) et declarer aussi les bon valeurs pour connecter a la DB 
- permission au repertoires : il faut creer attribuer les droits a tous les repertoires que nous avons declaré dans nos fichiers YAML avant d'executer ( dans les hostpath ) avec la commande suivante :
```
sudo mkdir -p /data_k8s/pgadmin4
sudo chmod -R 777 /data_k8s/pgadmin4
```
tous ces manifestes sont classé dans un dossier pg_admin , pour lancer notre site pg_admin , il faut ce rendre dans le dossier pg_admin sur le terminal et executer la commande suivante : 
```
kubectl apply -f .
```  

le résultat doit être comme suit : 

![image](https://github.com/adda213/mini-projet-K8S/assets/123883398/291f7651-10d4-44dd-ac65-3e5978459b71)
