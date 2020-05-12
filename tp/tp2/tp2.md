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

 ## Gestions des utilisateurs et des groupes
