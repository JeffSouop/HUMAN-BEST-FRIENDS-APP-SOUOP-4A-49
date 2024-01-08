# Human best Friend app
## Déploiement mannuel 
- Création et déploiement des conteneurs.
- Creation des images en buildant les fichiers Dockerfile de chacune des parties et pull des images si besoin.
- Creation des conteneurs a partir des images créées précedemment.
:warning: Démarrer les conteneurs avant toute autre manipulation :warning:
- Création des réseaux.
- Connecter les machines selon la configuration de la consigne.
- Tester en accédant aux applications depuis l'extérieur :
* - Taper "l'addresse IP de la machine hôte":"port sur lequel tourne l'application que l'on veut tester"  
## Automatisation de la création et du déploiement
- Création du fichier docker-compose.yml.
- Build du fichier avec la commande : docker compose build qui générera toute les images. 
- Création des conteneurs avec les images créées à l'aide docker compose up.
- Test des différents conteneurs de la même manière que pour la méthode manuelle.

###
https://docs.docker.com/compose/

