# Guide d'atelier pratique - Séance 3 : Gestion des processus et services sous Linux

---

## 🏢 **SCÉNARIO PROFESSIONNEL**

Vous êtes stagiaire administrateur système chez **DevCorp**, une entreprise qui développe des applications web. Votre responsable vous confie la gestion d'un serveur de développement Ubuntu qui présente des problèmes de performance. L'équipe de développeurs se plaint que le serveur est lent et que certains services s'arrêtent de manière imprévisible. Votre mission : diagnostiquer les problèmes, optimiser le système, automatiser la maintenance et créer un service de surveillance personnalisé pour éviter que ces problèmes se reproduisent.

---

## 📋 **MATÉRIEL NÉCESSAIRE**

- **Machine virtuelle** : Ubuntu Server 22.04 LTS ou Ubuntu Desktop
- **Accès administrateur** : Compte utilisateur avec droits sudo
- **Éditeur de texte** : nano, vim ou gedit
- **Terminal** : Accès à la ligne de commande

---

## 🎯 **ÉTAPE 1 : Diagnostic et nettoyage du système (15 min)**

### **Objectif :**
Analyser les processus en cours, identifier ceux qui consomment trop de ressources et nettoyer le système.

### **1.1 - Analyser la charge système**
```bash
# Voir l'état général du système
top
# Appuyer sur 'q' pour quitter après observation

# Alternative plus lisible
htop
# Si htop n'est pas installé :
sudo apt update && sudo apt install htop -y
htop
```

### **1.2 - Identifier les processus gourmands**
```bash
# Lister tous les processus avec leur consommation
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10

# Chercher des processus spécifiques suspects
ps aux | grep firefox
ps aux | grep -i zombie
```

### **1.3 - Simuler et nettoyer des processus problématiques**
```bash
# Créer un processus gourmand pour la simulation (NE PAS FAIRE EN PRODUCTION)
yes > /dev/null &
echo "PID du processus test : $!"

# Identifier et terminer proprement
pgrep yes
pkill yes

# Vérifier qu'il n'y a plus de processus yes
pgrep yes
```

### **1.4 - Vérifier les services système**
```bash
# Voir les services en échec
systemctl --failed

# Voir les services les plus gourmands
systemctl list-units --type=service --state=running
```

### **Résultat attendu :**
- Identification des processus gourmands en CPU/mémoire
- Nettoyage réussi des processus inutiles
- Liste des services en fonctionnement
- Aucun service en échec critique

### **Test de vérification :**
```bash
# Le système doit être plus fluide
top
# Doit montrer une charge CPU < 20% et pas de processus 'yes'
```

---

## ⏰ **ÉTAPE 2 : Automatisation des tâches de maintenance (20 min)**

### **Objectif :**
Mettre en place des tâches automatisées pour la maintenance préventive du système.

### **2.1 - Créer un script de surveillance d'espace disque**
```bash
# Créer le répertoire pour nos scripts
sudo mkdir -p /opt/devcorp/scripts

# Créer le script de surveillance
sudo nano /opt/devcorp/scripts/check_disk.sh
```

**Contenu du script :**
```bash
#!/bin/bash
# Script de surveillance d'espace disque - DevCorp

LOG_FILE="/var/log/devcorp_disk.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Vérifier l'espace disque du répertoire racine
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')

echo "[$DATE] Utilisation disque racine: ${DISK_USAGE}%" >> $LOG_FILE

if [ $DISK_USAGE -gt 80 ]; then
    echo "[$DATE] ALERTE: Espace disque critique (${DISK_USAGE}%)" >> $LOG_FILE
    echo "Espace disque critique sur $(hostname): ${DISK_USAGE}%" | mail -s "Alerte DevCorp" admin@devcorp.local 2>/dev/null || echo "[$DATE] Mail non configuré" >> $LOG_FILE
else
    echo "[$DATE] Espace disque OK (${DISK_USAGE}%)" >> $LOG_FILE
fi
```

```bash
# Rendre le script exécutable
sudo chmod +x /opt/devcorp/scripts/check_disk.sh

# Tester le script
sudo /opt/devcorp/scripts/check_disk.sh

# Vérifier le résultat
sudo cat /var/log/devcorp_disk.log
```

### **2.2 - Créer un script de nettoyage automatique**
```bash
# Créer le script de nettoyage
sudo nano /opt/devcorp/scripts/cleanup.sh
```

**Contenu du script :**
```bash
#!/bin/bash
# Script de nettoyage automatique - DevCorp

LOG_FILE="/var/log/devcorp_cleanup.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "[$DATE] Début du nettoyage automatique" >> $LOG_FILE

# Nettoyer les fichiers temporaires vieux de plus de 7 jours
TEMP_FILES=$(find /tmp -type f -mtime +7 2>/dev/null | wc -l)
find /tmp -type f -mtime +7 -delete 2>/dev/null
echo "[$DATE] Supprimé $TEMP_FILES fichiers temporaires" >> $LOG_FILE

# Nettoyer les logs anciens (garder 30 jours)
LOG_FILES=$(find /var/log -name "*.log" -mtime +30 2>/dev/null | wc -l)
sudo find /var/log -name "*.log" -mtime +30 -delete 2>/dev/null
echo "[$DATE] Supprimé $LOG_FILES anciens fichiers de log" >> $LOG_FILE

# Vider la corbeille des utilisateurs
for USER_HOME in /home/*; do
    if [ -d "$USER_HOME/.local/share/Trash" ]; then
        rm -rf "$USER_HOME/.local/share/Trash/files/*" 2>/dev/null
        rm -rf "$USER_HOME/.local/share/Trash/info/*" 2>/dev/null
    fi
done
echo "[$DATE] Corbeilles vidées" >> $LOG_FILE

echo "[$DATE] Nettoyage terminé" >> $LOG_FILE
```

```bash
# Rendre exécutable et tester
sudo chmod +x /opt/devcorp/scripts/cleanup.sh
sudo /opt/devcorp/scripts/cleanup.sh

# Vérifier le résultat
sudo cat /var/log/devcorp_cleanup.log
```

### **2.3 - Programmer les tâches avec cron**
```bash
# Éditer la crontab root pour les tâches système
sudo crontab -e
```

**Ajouter ces lignes :**
```bash
# Surveillance espace disque toutes les heures
0 * * * * /opt/devcorp/scripts/check_disk.sh

# Nettoyage automatique tous les dimanches à 2h du matin
0 2 * * 0 /opt/devcorp/scripts/cleanup.sh

# Sauvegarde des logs DevCorp tous les jours à 1h du matin
0 1 * * * cp /var/log/devcorp_*.log /opt/devcorp/backup/ 2>/dev/null
```

```bash
# Créer le répertoire de sauvegarde
sudo mkdir -p /opt/devcorp/backup

# Vérifier la crontab
sudo crontab -l
```

### **2.4 - Tester les tâches programmées**
```bash
# Programmer une tâche de test dans 2 minutes
at now + 2 minutes
# Dans l'interface at, taper :
echo "Test automatisation DevCorp - $(date)" >> /tmp/test_devcorp.txt
# Appuyer sur Ctrl+D

# Vérifier les tâches at programmées
atq

# Attendre 2 minutes puis vérifier
cat /tmp/test_devcorp.txt
```

### **Résultat attendu :**
- Scripts de maintenance créés et fonctionnels
- Tâches cron programmées correctement
- Logs de surveillance générés
- Test d'automatisation réussi

### **Test de vérification :**
```bash
# Vérifier les crons actifs
sudo crontab -l

# Vérifier que les scripts existent et sont exécutables
ls -la /opt/devcorp/scripts/
```

---

## 🚀 **ÉTAPE 3 : Création d'un service de surveillance personnalisé (20 min)**

### **Objectif :**
Créer un service systemd personnalisé qui surveille l'état du système en continu.

### **3.1 - Créer le script du service de surveillance**
```bash
# Créer le script principal du service
sudo nano /opt/devcorp/scripts/system_monitor.sh
```

**Contenu du script :**
```bash
#!/bin/bash
# Service de surveillance système DevCorp

LOG_FILE="/var/log/devcorp_monitor.log"
PID_FILE="/var/run/devcorp_monitor.pid"

# Enregistrer le PID
echo $$ > $PID_FILE

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> $LOG_FILE
}

log_message "Service de surveillance DevCorp démarré (PID: $$)"

while true; do
    # Surveillance de la charge CPU
    CPU_LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
    CPU_PERCENT=$(echo "$CPU_LOAD * 100" | bc 2>/dev/null || echo "0")
    
    # Surveillance mémoire
    MEM_USAGE=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    
    # Surveillance processus
    PROCESS_COUNT=$(ps aux | wc -l)
    
    # Log des métriques
    log_message "CPU: ${CPU_LOAD}, RAM: ${MEM_USAGE}%, Processus: ${PROCESS_COUNT}"
    
    # Alertes si nécessaire
    if (( $(echo "$MEM_USAGE > 90" | bc -l) )); then
        log_message "ALERTE: Utilisation mémoire critique (${MEM_USAGE}%)"
    fi
    
    if [ $PROCESS_COUNT -gt 200 ]; then
        log_message "ALERTE: Nombre de processus élevé ($PROCESS_COUNT)"
    fi
    
    # Attendre 30 secondes
    sleep 30
done
```

```bash
# Rendre le script exécutable
sudo chmod +x /opt/devcorp/scripts/system_monitor.sh

# Installer bc pour les calculs (si nécessaire)
sudo apt install bc -y
```

### **3.2 - Créer le fichier service systemd**
```bash
# Créer le fichier service
sudo nano /etc/systemd/system/devcorp-monitor.service
```

**Contenu du fichier service :**
```ini
[Unit]
Description=Service de surveillance système DevCorp
Documentation=https://devcorp.local/docs/monitoring
After=network.target
Wants=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/opt/devcorp/scripts/system_monitor.sh
ExecStop=/bin/kill -TERM $MAINPID
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=devcorp-monitor

# Sécurité
PrivateTmp=true
ProtectHome=true
NoNewPrivileges=true

# Fichiers et répertoires
WorkingDirectory=/opt/devcorp
PIDFile=/var/run/devcorp_monitor.pid

[Install]
WantedBy=multi-user.target
```

### **3.3 - Activer et démarrer le service**
```bash
# Recharger systemd pour prendre en compte le nouveau service
sudo systemctl daemon-reload

# Activer le service au démarrage
sudo systemctl enable devcorp-monitor.service

# Démarrer le service
sudo systemctl start devcorp-monitor.service

# Vérifier l'état du service
systemctl status devcorp-monitor.service
```

### **3.4 - Tester la robustesse du service**
```bash
# Voir les logs du service en temps réel
sudo journalctl -u devcorp-monitor.service -f &

# Dans un autre terminal, tester le redémarrage automatique
sudo systemctl stop devcorp-monitor.service
sudo systemctl start devcorp-monitor.service

# Tester le crash et redémarrage automatique
sudo pkill -f system_monitor.sh

# Attendre 10 secondes et vérifier que le service redémarre
sleep 15
systemctl status devcorp-monitor.service
```

### **Résultat attendu :**
- Service systemd créé et actif
- Logs de surveillance générés automatiquement
- Redémarrage automatique en cas d'arrêt
- Service activé au démarrage du système

### **Test de vérification :**
```bash
# Le service doit être actif
systemctl is-active devcorp-monitor.service

# Des logs doivent être générés
sudo tail -5 /var/log/devcorp_monitor.log

# Le service doit être activé au démarrage
systemctl is-enabled devcorp-monitor.service
```

---

## 📊 **ÉTAPE 4 : Diagnostic et analyse des logs (5 min)**

### **Objectif :**
Analyser les journaux système et résoudre un problème simulé.

### **4.1 - Analyser les logs système**
```bash
# Voir les erreurs récentes du système
sudo journalctl -p err --since "1 hour ago"

# Voir les logs de notre service
sudo journalctl -u devcorp-monitor.service --since "10 minutes ago"

# Analyser les connexions système
sudo journalctl -u ssh --since "today"

# Voir les dernières actions sudo
sudo journalctl | grep sudo | tail -10
```

### **4.2 - Résoudre un problème simulé**
```bash
# Simuler un problème : créer un service défaillant
sudo nano /tmp/broken_service.sh
```

**Contenu :**
```bash
#!/bin/bash
echo "Service démarré"
sleep 5
echo "Erreur simulée" >&2
exit 1
```

```bash
# Créer un service défaillant
sudo chmod +x /tmp/broken_service.sh

sudo tee /etc/systemd/system/broken-test.service << EOF
[Unit]
Description=Service de test défaillant

[Service]
Type=oneshot
ExecStart=/tmp/broken_service.sh
EOF

# Recharger et démarrer le service défaillant
sudo systemctl daemon-reload
sudo systemctl start broken-test.service

# Diagnostiquer le problème
systemctl status broken-test.service
sudo journalctl -u broken-test.service

# Nettoyer après diagnostic
sudo systemctl disable broken-test.service
sudo rm /etc/systemd/system/broken-test.service
sudo rm /tmp/broken_service.sh
sudo systemctl daemon-reload
```

### **Résultat attendu :**
- Analyse réussie des logs système
- Identification et diagnostic du service défaillant
- Nettoyage après résolution du problème

---

## 📝 **LIVRABLE : RAPPORT D'ATELIER**

Créez un fichier texte nommé `rapport_atelier_processus_[VOTRE_NOM].txt` contenant :

### **1. Résumé exécutif (3-4 lignes)**
Décrivez en quelques phrases ce que vous avez accompli pendant cet atelier.

### **2. Diagnostic initial du système**
```
- Processus les plus gourmands identifiés :
- Services en échec trouvés :
- Actions de nettoyage effectuées :
```

### **3. Configuration de l'automatisation**
```
- Scripts créés :
  * /opt/devcorp/scripts/check_disk.sh ✓/✗
  * /opt/devcorp/scripts/cleanup.sh ✓/✗
  * /opt/devcorp/scripts/system_monitor.sh ✓/✗

- Tâches cron programmées :
  * Surveillance disque (toutes les heures) ✓/✗
  * Nettoyage hebdomadaire ✓/✗
  * Sauvegarde logs quotidienne ✓/✗
```

### **4. Service personnalisé**
```
- Service devcorp-monitor.service :
  * Créé ✓/✗
  * Démarré ✓/✗
  * Activé au boot ✓/✗
  * Redémarrage automatique testé ✓/✗
```

### **5. Difficultés rencontrées et solutions**
Décrivez les problèmes rencontrés et comment vous les avez résolus.

### **6. Commandes de vérification finale**
```bash
# Copier-coller le résultat de ces commandes :
systemctl status devcorp-monitor.service
sudo crontab -l
ls -la /opt/devcorp/scripts/
sudo tail -3 /var/log/devcorp_monitor.log
```

---

## ✅ **CHECKLIST D'AUTO-VALIDATION**

Cochez chaque élément vérifié :

**Diagnostic et nettoyage :**
- [ ] J'ai identifié les processus gourmands avec ps et top
- [ ] J'ai nettoyé les processus inutiles
- [ ] J'ai vérifié qu'aucun service critique n'est en échec
- [ ] Le système est plus fluide qu'au début

**Automatisation :**
- [ ] Mes 3 scripts sont créés et exécutables
- [ ] Les tâches cron sont programmées correctement
- [ ] Les logs de surveillance sont générés
- [ ] Le test d'automatisation avec 'at' a fonctionné

**Service personnalisé :**
- [ ] Le service devcorp-monitor est créé et actif
- [ ] Le service génère des logs de surveillance
- [ ] Le service redémarre automatiquement en cas d'arrêt
- [ ] Le service est activé pour démarrer au boot

**Analyse et diagnostic :**
- [ ] J'ai analysé les logs système avec journalctl
- [ ] J'ai diagnostiqué et résolu le problème simulé
- [ ] J'ai nettoyé les fichiers de test

**Livrable :**
- [ ] Mon rapport est complet et détaillé
- [ ] J'ai documenté les difficultés rencontrées
- [ ] Toutes les commandes de vérification sont incluses