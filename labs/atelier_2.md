# Guide d'atelier pratique - Séance 2 : Gestion des utilisateurs et des droits

## Scénario professionnel

Vous êtes administrateur système chez DataSecure, une entreprise spécialisée dans l'hébergement de données sensibles. Votre responsable vous demande de configurer un nouveau serveur pour un projet collaboratif impliquant plusieurs équipes : développeurs, testeurs et administrateurs. Votre mission est de créer les comptes utilisateurs nécessaires, d'organiser les groupes et de mettre en place une structure de permissions adaptée aux besoins de collaboration tout en garantissant la sécurité des données.

## Objectifs de l'atelier
- Créer des utilisateurs et des groupes selon les besoins du projet
- Configurer une structure de répertoires avec permissions appropriées
- Mettre en place des permissions spéciales (SGID, sticky bit)
- Configurer sudo pour certains utilisateurs
- Tester et valider les permissions

## Matériel nécessaire
- Une machine virtuelle Linux déjà installée (Ubuntu Server ou similaire)
- Connexion en tant qu'utilisateur avec droits sudo

## Étape 1 : Création des utilisateurs et des groupes (15 min)

1. Créer les groupes suivants :
   ```bash
   sudo groupadd devteam      # Équipe de développement
   sudo groupadd testteam     # Équipe de test
   sudo groupadd adminteam    # Équipe d'administration
   sudo groupadd projectmgrs  # Gestionnaires de projet
   ```

2. Créer les utilisateurs suivants avec bash comme shell par défaut :
   ```bash
   # Développeurs
   sudo useradd -m -s /bin/bash -c "Yassine Developer" yassine
   sudo useradd -m -s /bin/bash -c "Issam Developer" issam
   
   # Testeurs
   sudo useradd -m -s /bin/bash -c "Imad Tester" imad
   sudo useradd -m -s /bin/bash -c "Jamal Tester" jamal
   
   # Administrateurs
   sudo useradd -m -s /bin/bash -c "Mohammed Admin" mohammed
   
   # Chef de projet
   sudo useradd -m -s /bin/bash -c "Hafid Manager" hafid
   ```

3. Définir des mots de passe pour tous les utilisateurs (pour cet exercice, utilisez le même mot de passe pour tous) :
   ```bash
   sudo passwd yassine
   sudo passwd issam
   sudo passwd imad
   sudo passwd jamal
   sudo passwd mohammed
   sudo passwd hafid
   ```

4. Ajouter les utilisateurs aux groupes appropriés :
   ```bash
   # Groupes primaires
   sudo usermod -g devteam yassine
   sudo usermod -g devteam issam
   sudo usermod -g testteam imad
   sudo usermod -g testteam jamal
   sudo usermod -g adminteam mohammed
   sudo usermod -g projectmgrs hafid
   
   # Groupes secondaires
   sudo usermod -aG projectmgrs yassine  # Yassine est aussi chef de projet
   sudo usermod -aG adminteam hafid    # Hafid a des droits d'admin
   ```

5. Vérifier la configuration :
   ```bash
   getent group devteam testteam adminteam projectmgrs
   id yassine
   id hafid
   ```

**Point de validation** : Tous les utilisateurs et groupes sont créés et configurés correctement. Vous devriez voir les appartenances aux groupes dans la sortie des commandes `getent` et `id`.

## Étape 2 : Configuration de la structure de répertoires et permissions (15 min)

1. Créer la structure de répertoires du projet :
   ```bash
   sudo mkdir -p /opt/datasecure/{development,testing,production,shared}
   ```

2. Définir les propriétaires et groupes appropriés :
   ```bash
   sudo chown root:devteam /opt/datasecure/development
   sudo chown root:testteam /opt/datasecure/testing
   sudo chown root:adminteam /opt/datasecure/production
   sudo chown root:projectmgrs /opt/datasecure/shared
   ```

3. Configurer les permissions de base :
   ```bash
   # Développeurs peuvent tout faire dans development
   sudo chmod 770 /opt/datasecure/development
   
   # Testeurs peuvent tout faire dans testing
   sudo chmod 770 /opt/datasecure/testing
   
   # Seuls les admins peuvent tout faire dans production
   sudo chmod 770 /opt/datasecure/production
   
   # Tous les membres du projet peuvent accéder au répertoire shared
   sudo chmod 775 /opt/datasecure/shared
   ```

4. Configurer le SGID sur les répertoires pour que les nouveaux fichiers héritent du groupe :
   ```bash
   sudo chmod g+s /opt/datasecure/development
   sudo chmod g+s /opt/datasecure/testing
   sudo chmod g+s /opt/datasecure/production
   sudo chmod g+s /opt/datasecure/shared
   ```

5. Ajouter le sticky bit sur le répertoire shared :
   ```bash
   sudo chmod +t /opt/datasecure/shared
   ```

6. Vérifier la configuration :
   ```bash
   ls -la /opt/datasecure/
   ```
   Vous devriez voir des permissions ressemblant à ceci :
   ```
   drwxrws---  2 root devteam      4096 mai 17 10:30 development
   drwxrws---  2 root testteam     4096 mai 17 10:30 production
   drwxrws---  2 root adminteam    4096 mai 17 10:30 testing
   drwxrwst--  2 root projectmgrs  4096 mai 17 10:30 shared
   ```

**Point de validation** : Les répertoires sont créés avec les bonnes permissions, propriétaires et groupes. Vérifiez bien la présence du 's' (SGID) et du 't' (sticky bit) dans les permissions.

## Étape 3 : Configuration de sudo pour les utilisateurs (10 min)

1. Créer un fichier de configuration sudo pour le groupe adminteam :
   ```bash
   sudo visudo -f /etc/sudoers.d/adminteam
   ```
   
   Ajouter la ligne suivante :
   ```
   %adminteam ALL=(ALL:ALL) ALL
   ```

2. Créer un fichier de configuration sudo pour le groupe projectmgrs avec des permissions limitées :
   ```bash
   sudo visudo -f /etc/sudoers.d/projectmgrs
   ```
   
   Ajouter les lignes suivantes :
   ```
   # Les chefs de projet peuvent gérer les utilisateurs mais pas modifier les fichiers système
   %projectmgrs ALL=(ALL:ALL) /usr/sbin/useradd, /usr/sbin/usermod, /usr/sbin/userdel, /usr/bin/passwd
   ```

3. Vérifier la configuration :
   ```bash
   sudo -l -U mohammed
   sudo -l -U hafid
   ```

**Point de validation** : Les règles sudo sont correctement configurées pour les groupes spécifiés. Mohammed devrait avoir tous les droits sudo, tandis que Hafid ne devrait pouvoir exécuter que les commandes liées à la gestion des utilisateurs.

## Étape 4 : Test et validation des permissions (20 min)

1. Créer des fichiers de test en tant que différents utilisateurs :
   ```bash
   # Se connecter en tant que yassine
   sudo su - yassine
   touch /opt/datasecure/development/yassine_dev.txt
   touch /opt/datasecure/shared/yassine_shared.txt
   echo "Contenu créé par Yassine" > /opt/datasecure/development/yassine_dev.txt
   exit
   
   # Se connecter en tant que imad
   sudo su - imad
   touch /opt/datasecure/testing/imad_test.txt
   touch /opt/datasecure/shared/imad_shared.txt
   echo "Contenu créé par Imad" > /opt/datasecure/testing/imad_test.txt
   exit
   
   # Se connecter en tant que mohammed
   sudo su - mohammed
   touch /opt/datasecure/production/mohammed_prod.txt
   touch /opt/datasecure/shared/eve_shared.txt
   echo "Contenu créé par Mohammed" > /opt/datasecure/production/mohammed_prod.txt
   exit
   ```

2. Tester les restrictions d'accès :
   ```bash
   # Yassine ne devrait pas pouvoir écrire dans production
   sudo su - yassine
   touch /opt/datasecure/production/yassine_unauthorized.txt
   # Devrait afficher "Permission denied"
   
   # Vérifier qu'Yassine peut lire le fichier de Imad dans testing
   cat /opt/datasecure/testing/imad_test.txt
   # Devrait fonctionner et afficher le contenu
   
   # Essayer de supprimer le fichier d'Mohammed dans shared
   rm /opt/datasecure/shared/eve_shared.txt
   # Devrait échouer à cause du sticky bit
   exit
   
   # Imad ne devrait pas pouvoir supprimer le fichier d'Yassine dans shared
   sudo su - imad
   rm /opt/datasecure/shared/yassine_shared.txt
   # Devrait échouer à cause du sticky bit
   exit
   
   # Mohammed devrait pouvoir tout faire
   sudo su - mohammed
   echo "Test d'accès admin" > /opt/datasecure/development/admin_file.txt
   cat /opt/datasecure/development/yassine_dev.txt
   # Devrait fonctionner
   exit
   ```

3. Tester les privilèges sudo :
   ```bash
   # Hafid (chef de projet) devrait pouvoir créer un utilisateur
   sudo su - hafid
   sudo useradd -m -s /bin/bash -c "New User" newuser
   # Devrait fonctionner
   
   # Hafid ne devrait pas pouvoir installer des packages
   sudo apt update
   # Devrait échouer avec une erreur de permission
   exit
   
   # Mohammed devrait pouvoir exécuter n'importe quelle commande sudo
   sudo su - mohammed
   sudo apt update
   # Devrait fonctionner
   exit
   ```

4. Vérifier l'héritage des groupes pour les nouveaux fichiers :
   ```bash
   # Créer un nouveau fichier dans development
   sudo su - issam
   touch /opt/datasecure/development/test_inheritance.txt
   ls -l /opt/datasecure/development/test_inheritance.txt
   # Le groupe devrait être devteam
   exit
   ```

**Point de validation** : Toutes les permissions fonctionnent comme prévu et les restrictions sont correctement appliquées. Les utilisateurs peuvent accéder aux ressources appropriées et sont limités pour les autres.

## Livrable à remettre
Un document "gestion-utilisateurs-droits.txt" contenant :

1. Un tableau récapitulatif des utilisateurs, groupes et leurs rôles
   ```
   | Utilisateur | Groupe(s)              | Rôle                           |
   |-------------|------------------------|--------------------------------|
   | yassine       | devteam, projectmgrs   | Développeur et chef de projet  |
   | issam         | devteam                | Développeur                    |
   | imad       | testteam               | Testeur                        |
   | jamal        | testteam               | Testeur                        |
   | mohammed         | adminteam              | Administrateur système         |
   | hafid       | projectmgrs, adminteam | Manager avec droits limités    |
   ```

2. Un schéma de la structure des répertoires avec leurs permissions
   ```
   /opt/datasecure/
   ├── development/ (770, SGID, root:devteam)
   ├── testing/     (770, SGID, root:testteam)
   ├── production/  (770, SGID, root:adminteam)
   └── shared/      (775, SGID, sticky bit, root:projectmgrs)
   ```

3. Les commandes principales que vous avez utilisées pour :
   - Créer des utilisateurs et groupes
   - Définir les permissions
   - Configurer sudo

4. Les résultats de vos tests et validations

5. Une réflexion sur l'importance des permissions dans un environnement multi-utilisateurs (5-10 lignes)

## Checklist d'auto-évaluation
- [ ] J'ai créé tous les utilisateurs et groupes demandés
- [ ] J'ai configuré correctement les répertoires avec les permissions appropriées
- [ ] J'ai appliqué les bits spéciaux (SGID, sticky bit) correctement
- [ ] J'ai configuré sudo pour les groupes spécifiés avec les bonnes restrictions
- [ ] J'ai testé et validé que les permissions fonctionnent comme prévu
- [ ] J'ai compris comment fonctionnent les permissions de base rwx
- [ ] Je sais comment les permissions s'appliquent différemment sur les fichiers et les répertoires
- [ ] Je comprends l'utilité des bits spéciaux (SUID, SGID, sticky bit)
- [ ] Je sais comment configurer des règles sudo spécifiques
- [ ] J'ai documenté mes actions et observations dans le livrable