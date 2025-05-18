# Atelier pratique : Premiers pas avec Linux

## Scénario professionnel

Vous venez d'être recruté(e) comme administrateur(trice) système junior dans la société TechInnovate. Votre première mission consiste à configurer un serveur Linux qui servira de plateforme de test pour l'équipe de développement. Vous devez mettre en place l'environnement de base et documenter les étapes pour faciliter la formation des futurs administrateurs.

## Objectifs de l'atelier
- Installer une machine virtuelle Linux
- Prendre en main l'environnement terminal
- Explorer le système de fichiers
- Créer et manipuler des fichiers et répertoires
- Documenter les opérations réalisées

## Matériel nécessaire
- Un ordinateur avec VirtualBox ou VMware installé
- Image ISO d'Ubuntu Server 22.04 LTS
- Au moins 2 Go de RAM et 10 Go d'espace disque disponibles pour la VM

## Étape 1 : Installation de la machine virtuelle (15 min)

1. Démarrer VirtualBox et créer une nouvelle machine virtuelle
   - Nom : "UbuntuServer-[VotreNom]"
   - Type : Linux
   - Version : Ubuntu 64-bit
   - Mémoire RAM : 2048 MB
   - Disque dur : 10 GB, dynamiquement alloué

2. Configurer la VM :
   - Réseau : Mode Pont ou NAT
   - Périphériques : Ajouter l'ISO d'Ubuntu Server

3. Démarrer la VM et suivre l'assistant d'installation
   - Langue : Français
   - Disposition clavier : Français
   - Installation minimale (sans interface graphique)
   - Partitionnement automatique
   - Nom d'utilisateur : admin
   - Nom de la machine : techinnovate-srv01

**Point de validation** : VM installée et opérationnelle, vous pouvez vous connecter avec l'utilisateur créé.

## Étape 2 : Premiers pas dans le terminal (15 min)

1. Se connecter à la VM avec les identifiants configurés

2. Mettre à jour le système :
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

3. Installer quelques utilitaires essentiels :
   ```bash
   sudo apt install -y htop net-tools tree curl
   ```

4. Explorer les informations système :
   ```bash
   uname -a           # Informations sur le noyau
   lsb_release -a     # Informations sur la distribution
   free -h            # État de la mémoire
   df -h              # Utilisation de l'espace disque
   ip a               # Configuration réseau
   ```

5. Utiliser les commandes de base pour naviguer dans le système de fichiers :
   ```bash
   pwd                # Afficher le répertoire courant
   ls -la             # Lister les fichiers, y compris cachés
   cd /etc            # Se déplacer dans /etc
   ls -l              # Lister les fichiers
   cd ~               # Retourner dans le répertoire personnel
   ```

**Point de validation** : Capture d'écran montrant la sortie des commandes `uname -a`, `free -h` et `df -h`.

## Étape 3 : Exploration du système de fichiers (15 min)

1. Explorer la structure des répertoires principaux :
   ```bash
   ls -la /           # Racine du système de fichiers
   ls -la /etc        # Fichiers de configuration
   ls -la /var/log    # Journaux système
   ls -la /usr/bin    # Binaires et applications
   ```

2. Examiner les fichiers de configuration importants :
   ```bash
   cat /etc/hostname  # Nom de la machine
   cat /etc/hosts     # Table de résolution de noms locale
   cat /etc/fstab     # Configuration des points de montage
   ```

3. Observer les processus système :
   ```bash
   ps aux             # Liste de tous les processus
   top                # Monitoring des processus (q pour quitter)
   htop               # Version améliorée de top (q pour quitter)
   ```

4. Explorer les fichiers journaux :
   ```bash
   sudo tail -n 50 /var/log/syslog  # Dernières entrées du journal système
   ```

**Point de validation** : Créer un fichier texte qui liste 3 observations intéressantes que vous avez faites pendant l'exploration.

## Étape 4 : Manipulation de fichiers et création d'un script (15 min)

1. Créer une structure de répertoires pour le projet :
   ```bash
   mkdir -p ~/techinnovate/scripts ~/techinnovate/logs ~/techinnovate/docs
   ```

2. Créer un fichier d'information de base :
   ```bash
   echo "Serveur de test TechInnovate" > ~/techinnovate/docs/README.txt
   echo "Créé le $(date)" >> ~/techinnovate/docs/README.txt
   echo "Par : $USER" >> ~/techinnovate/docs/README.txt
   ```

3. Créer un script de surveillance système :
   ```bash
   nano ~/techinnovate/scripts/system_info.sh
   ```

4. Ajouter le contenu suivant au script :
   ```bash
   #!/bin/bash
   
   # Script de surveillance système pour TechInnovate
   # Créé dans le cadre de la formation d'administration système
   
   LOGFILE=~/techinnovate/logs/system_$(date +%Y%m%d).log
   
   echo "===== Rapport système du $(date) =====" >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "-- Informations système --" >> $LOGFILE
   uname -a >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "-- Uptime --" >> $LOGFILE
   uptime >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "-- Utilisation mémoire --" >> $LOGFILE
   free -h >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "-- Espace disque --" >> $LOGFILE
   df -h >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "-- Processus les plus gourmands en CPU --" >> $LOGFILE
   ps aux --sort=-%cpu | head -11 >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "-- Processus les plus gourmands en mémoire --" >> $LOGFILE
   ps aux --sort=-%mem | head -11 >> $LOGFILE
   echo "" >> $LOGFILE
   
   echo "Rapport généré dans $LOGFILE"
   ```

5. Rendre le script exécutable et le tester :
   ```bash
   chmod +x ~/techinnovate/scripts/system_info.sh
   ~/techinnovate/scripts/system_info.sh
   ```

6. Examiner le rapport généré :
   ```bash
   cat ~/techinnovate/logs/system_*.log
   ```

**Point de validation** : Script fonctionnel qui génère un rapport système complet.

## Livrable à remettre

Un document "premier-pas-linux.txt" contenant :

1. Le nom d'hôte et adresse IP de votre serveur
2. La liste des commandes explorées pendant l'atelier avec une brève description de chacune
3. Trois observations que vous avez faites sur la structure des répertoires Linux
4. Le contenu de votre script system_info.sh
5. Une réflexion personnelle sur ce que vous avez appris (5-10 lignes)

## Checklist d'auto-évaluation

- [ ] J'ai réussi à installer une VM Ubuntu Server fonctionnelle
- [ ] J'ai utilisé correctement les commandes de base de navigation et d'affichage
- [ ] J'ai exploré la structure des répertoires et compris leur rôle
- [ ] J'ai créé une structure de répertoires organisée pour mon projet
- [ ] J'ai écrit un script bash fonctionnel qui collecte des informations système
- [ ] J'ai documenté mes actions et observations
- [ ] Je comprends la différence entre utilisateur normal et superutilisateur (sudo)
- [ ] Je sais utiliser un éditeur de texte en ligne de commande (nano)
- [ ] J'ai généré le livrable demandé complet
- [ ] Je pourrais expliquer à quelqu'un d'autre les bases de ce que j'ai appris aujourd'hui