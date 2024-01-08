# Projet Virtualisation - Audrey Jordan SOUOP 4A Classe 49

## HUMAN BEST FRIEND APP

Ce projet vise à déployer et tester une application à l'aide de Docker

### Processus Manuel

1. **Création des Environnements Docker :**
   - Créez des environnements isolés pour exécuter les applications, utilisant Docker pour la gestion de conteneurs.

2. **Construction Manuelle des Images :**
   - Utilisez les Dockerfiles pour construire les images nécessaires, téléchargez les images complémentaires, puis créez les conteneurs correspondants.

3. **Lancement des Conteneurs :**
   - Assurez-vous de démarrer les conteneurs avant toute manipulation future pour garantir leur fonctionnement.

4. **Configuration du Réseau :**
   - Établissez des réseaux virtuels pour connecter et isoler les conteneurs, selon les besoins spécifiques du projet.

5. **Connectivité entre Conteneurs :**
   - Configurez les liens et les interactions entre les différents conteneurs conformément aux instructions spécifiées.

6. **Tests Manuels :**
   - Vérifiez le bon fonctionnement des conteneurs en accédant aux applications via "adresse IP de la machine hôte":"port de l'application à tester".

### Processus Automatisé

1. **Utilisation de docker-compose :**
   - Création du fichier docker-compose.yml détaillant la configuration des services, incluant images, réseaux, volumes et configurations nécessaires.

2. **Construction Automatisée des Images :**
   - Exécution de `docker-compose build` pour construire automatiquement les images des services définis dans docker-compose.yml, en utilisant les Dockerfiles et configurations spécifiés.

3. **Déploiement Automatisé :**
   - Lancement des conteneurs avec `docker-compose up`, utilisant les images nouvellement construites pour créer les services selon les spécifications définies.

4. **Tests Automatisés :**
   - Vérification du bon fonctionnement des conteneurs, accédant à chaque service via l'adresse IP de la machine hôte suivie du port défini dans la configuration, de manière similaire au processus manuel.
