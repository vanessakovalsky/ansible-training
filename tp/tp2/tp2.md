# Utilisation des modules principaux d'ansible

Ce TP va nous permettre de manipuler avec la ligne de commande les principaux modules d'ansible : fichier, utilisateur, paquet, service, shell.
Nous allons réaliser les actions suivantes en utilisant les modules ansibles :
 - Créer les groupes rennes, lille et paris 
 - Créer un utilisateur core 
 - Modifier l’utilisateur core pour qu’il ait l’UID 10000 
 - Modifier l’utilisateur core pour qu’il soit dans le groupe rennes 
 - Installer le logiciel tree 
 - Stopper le service crond 
 - Désactiver le service atd 
 - créer un fichier vide /tmp/test avec les droits 644
 - Mettre à jour le système sur la VM cliente 
 - Redémarrer la VM cliente 

## Pré-requis
- Ce TP suppose que vous avez un environnement installé (voir TP1)
- L'ensemble des commandes est à réaliser dans la machine de gestion qui contient ansible
```
docker exec –i -t <containerID> bash
```

 ## Gestions des utilisateurs et des groupes
- On commence par créer les groupes
```
ansible testservers -m group -a "name=rennes" 
ansible testservers -m group -a "name=lille gid=1005"  
ansible testservers -m group -a "name=paris gid=1010" 
```
- On crée un utilisateur :
```
ansible testservers -m user -a "name=core" 
```
- On lui assigne l'UID 10000
```
ansible testservers -m user -a "name=core uid=10000" 
```
- On ajoute l'utilisateur au groupe "rennes":
```
ansible testservers -m user -a "name=core groups=rennes" 
```

## Installer un logiciel :
- Comme il s'agit de deux versions différentes de linux, on va nommer les hosts, modifier le fichier /etc/ansible/hosts : 
```
[testservers] 
host01 ansible_host=<IPmachinehost>
host02 ansible_host=<IPmachinehost2>
host03 ansible_host=<IPmachinehost3>
```
- Pour le serveur centos (host02) on utilise yum 
```
 ansible host02 -m yum -a "name=tree"
 ```
- Pour le serveur debian (host01) et ubuntu on utilise apt :
```
 ansible host01 -m apt -a "name=tree"
 ansible host03 -m apt -a "name=tree"
```

## Création d'un fichier 
- On crée un fichier vide et on lui met des droits 0644 :
```
 ansible testservers -m copy -a "content='' dest=/tmp/test force=no mode=644"
```
- Comment assigner le groupe lille comme propriétaire (groupe) du fichier ?

## Désactiver les services :
- On stoppe le service crond :
```
ansible testservers -m service -a "name=crond state=stopped"
```
- On désactive le service auditd
```
ansible testservers -m service -a "name=auditd enabled=no"
```

## Mise à jour du système 
- On met à jour l'ensemble des paquets de l'hote :
```
 ansible host02 -m yum -a "name=* state=latest"
```
- Chercher comment faire la même chose avec le module apt pour host01 et host03

## Rédémarrer la machine hôte
- On redémarre les machines hôtes :
```
ansible testservers -m shell -a "name=reboot"
```

-> Vous savez maintenant manipulez les principaux modules de ansible, n'hésitez pas à aller en découvrir d'autres sur :
https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html
