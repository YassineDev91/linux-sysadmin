# Atelier Pratique : Scripts shell et automatisation
## Script de sauvegarde intelligent - Durée : 1 heure

---

## 🏢 **SCÉNARIO PROFESSIONNEL**

Vous êtes stagiaire administrateur système chez **DataFlow**, une entreprise de traitement de données. Votre mission : créer un script de sauvegarde intelligent qui évoluera étape par étape. L'objectif est de maîtriser les concepts fondamentaux de Bash en construisant progressivement un outil professionnel de sauvegarde.

**Mission finale :** Script qui sauvegarde plusieurs dossiers, vérifie l'espace disponible, fait de la rotation automatique et analyse ses propres logs !

---

##  **MATÉRIEL NÉCESSAIRE**

- **Machine virtuelle** : Ubuntu 22.04 LTS
- **Éditeur** : nano (simple pour débuter)
- **Terminal** : Accès ligne de commande
- **Espace de travail** : `~/dataflow-backup`

---

## **ÉTAPE 1 : Variables et affichage de base**

### **Objectif : Maîtriser les variables Bash**

### **1.1 - Préparation**
```bash
# Créer l'espace de travail
mkdir -p ~/dataflow-backup
cd ~/dataflow-backup

# Créer un fichier de test à sauvegarder
mkdir -p test-data
echo "Fichier de test DataFlow" > test-data/important.txt
echo "Configuration système" > test-data/config.txt
```

### **1.2 - Premier script avec variables**

**Créer le script :**
```bash
nano backup.sh
```

**Contenu - Version 1 :**
```bash
#!/bin/bash
#
# Script de sauvegarde DataFlow - Étape 0
#

echo "=== Script de sauvegarde DataFlow ==="

# Variables de base (ATTENTION : pas d'espace autour du =)
nom_entreprise="DataFlow"
version_script="1.0"
date_actuelle=$(date +"%Y-%m-%d %H:%M:%S")
utilisateur_actuel=$USER

# Variables de configuration
dossier_source="test-data"
dossier_destination="backups"
nom_sauvegarde="backup_$(date +%Y%m%d_%H%M%S).tar.gz"

# Affichage des informations
echo "Entreprise : $nom_entreprise"
echo "Version du script : $version_script"
echo "Date : $date_actuelle"
echo "Utilisateur : $utilisateur_actuel"
echo ""

echo "Configuration de sauvegarde :"
echo "  Source : $dossier_source"
echo "  Destination : $dossier_destination"
echo "  Nom de fichier : $nom_sauvegarde"
echo ""

# Création simple d'une sauvegarde
echo " Création de la sauvegarde..."
mkdir -p "$dossier_destination"

# Commande tar expliquée :
# -c : créer une archive
# -z : compresser avec gzip
# -f : spécifier le nom du fichier
tar -czf "$dossier_destination/$nom_sauvegarde" "$dossier_source"

echo " Sauvegarde créée : $dossier_destination/$nom_sauvegarde"
echo "Script terminé !"
```

**Tester :**
```bash
chmod +x backup.sh
./backup.sh

# Vérifier le résultat
ls -la backups/
```

### **Point de contrôle Étape 1**
- Variables correctement déclarées et utilisées
- Sauvegarde tar.gz créée
- Affichage informatif fonctionnel

---

##  **ÉTAPE 2 : Conditions et tests (15 min)**

### **Objectif : Ajouter de l'intelligence avec les conditions**

### **2.1 - Amélioration avec vérifications**

**Modifier le script (garder le début, ajouter les vérifications) :**
```bash
nano backup.sh
```

**Remplacer la partie "Création simple" par cette version :**
```bash
# === NOUVELLE PARTIE : Vérifications avec conditions ===

# Vérification 1 : Le dossier source existe-t-il ?
echo "Vérifications préalables..."

if [ -d "$dossier_source" ]; then
    echo "Dossier source trouvé : $dossier_source"
else
    echo "  ERREUR : Dossier source manquant : $dossier_source"
    echo "  Script arrêté."
    exit 1  # Sortir du script avec code d'erreur
fi

# Vérification 2 : Y a-t-il des fichiers à sauvegarder ?
nombre_fichiers=$(find "$dossier_source" -type f | wc -l)
echo "  📁 Nombre de fichiers trouvés : $nombre_fichiers"

if [ $nombre_fichiers -eq 0 ]; then
    echo "  ATTENTION : Aucun fichier à sauvegarder !"
    echo "  Script arrêté."
    exit 1
else
    echo "   Fichiers prêts pour sauvegarde"
fi

# Vérification 3 : Espace disque disponible
echo "  Vérification de l'espace disque..."
espace_libre_kb=$(df . | awk 'NR==2 {print $4}')  # Espace libre en KB
espace_libre_mb=$((espace_libre_kb / 1024))        # Conversion en MB

echo "  Espace libre : ${espace_libre_mb} MB"

if [ $espace_libre_mb -lt 100 ]; then
    echo "  ATTENTION : Peu d'espace libre (moins de 100 MB)"
    echo "  Sauvegarde quand même..."
else
    echo "  Espace disque suffisant"
fi

echo ""

# Création du dossier de destination
if [ ! -d "$dossier_destination" ]; then
    echo "Création du dossier de destination..."
    mkdir -p "$dossier_destination"
    echo "  Dossier créé : $dossier_destination"
fi

# Création de la sauvegarde (maintenant qu'on a tout vérifié)
echo " Création de la sauvegarde..."
if tar -czf "$dossier_destination/$nom_sauvegarde" "$dossier_source" 2>/dev/null; then
    # Calculer la taille du fichier créé
    taille_fichier=$(du -h "$dossier_destination/$nom_sauvegarde" | cut -f1)
    echo "  Sauvegarde créée avec succès !"
    echo "  Fichier : $nom_sauvegarde"
    echo "  Taille : $taille_fichier"
else
    echo "  ERREUR lors de la création de la sauvegarde !"
    exit 1
fi

echo ""
echo " Script terminé avec succès !"
```

**Tester les différents cas :**
```bash
# Test normal
./backup.sh

# Test avec dossier source manquant
mv test-data test-data-backup
./backup.sh
mv test-data-backup test-data

# Revérifier que ça marche
./backup.sh
```

### **Point de contrôle Étape 1**
- Vérifications avant sauvegarde
- Gestion des erreurs avec exit
- Tests de fichiers et calculs numériques

---

## **ÉTAPE 3 : Boucles et gestion multiple (15 min)**

### **Objectif : Sauvegarder plusieurs dossiers et gérer la rotation**

### **3.1 - Modification pour gérer plusieurs sources**

**Remplacer la section configuration par :**
```bash
# === Variables de configuration améliorées ===
nom_entreprise="DataFlow"
version_script="2.0"
date_actuelle=$(date +"%Y-%m-%d %H:%M:%S")
utilisateur_actuel=$USER

# Liste des dossiers à sauvegarder (séparés par des espaces)
dossiers_source="test-data $HOME/.bashrc"  # On ajoute .bashrc comme exemple
dossier_destination="backups"
nom_sauvegarde="backup_$(date +%Y%m%d_%H%M%S).tar.gz"
max_sauvegardes=3  # Garder seulement 3 sauvegardes

echo "Entreprise : $nom_entreprise"
echo "Version du script : $version_script"
echo "Date : $date_actuelle"
echo "Utilisateur : $utilisateur_actuel"
echo ""

echo "Configuration de sauvegarde :"
echo "  Sources : $dossiers_source"
echo "  Destination : $dossier_destination"
echo "  Nom de fichier : $nom_sauvegarde"
echo "  Rétention : $max_sauvegardes sauvegardes"
echo ""
```

**Remplacer la section vérifications par :**
```bash
# === Vérifications avec boucles ===
echo "Vérification des sources..."

sources_valides=""  # Liste des sources qui existent vraiment
total_fichiers=0

# Boucle pour vérifier chaque source
for source in $dossiers_source; do
    echo "  Test de : $source"
    
    if [ -e "$source" ]; then  # -e teste fichier OU dossier
        echo "    Trouvé"
        sources_valides="$sources_valides $source"
        
        # Compter les fichiers (différent selon si c'est un fichier ou dossier)
        if [ -f "$source" ]; then
            fichiers_source=1  # C'est un fichier
        else
            fichiers_source=$(find "$source" -type f 2>/dev/null | wc -l)
        fi
        
        echo "    Fichiers : $fichiers_source"
        total_fichiers=$((total_fichiers + fichiers_source))
    else
        echo "    Introuvable (ignoré)"
    fi
done

echo ""
echo " Résumé :"
echo "  Sources valides :$sources_valides"
echo "  Total fichiers : $total_fichiers"

if [ $total_fichiers -eq 0 ]; then
    echo "  Aucun fichier à sauvegarder !"
    exit 1
fi

# Vérification espace disque (identique à l'étape 1)
echo "  Vérification de l'espace disque..."
espace_libre_kb=$(df . | awk 'NR==2 {print $4}')
espace_libre_mb=$((espace_libre_kb / 1024))
echo "  Espace libre : ${espace_libre_mb} MB"

if [ $espace_libre_mb -lt 100 ]; then
    echo "  Peu d'espace libre"
fi

echo ""
```

**Ajouter la gestion de rotation avant la création :**
```bash
# === Rotation des anciennes sauvegardes ===
if [ ! -d "$dossier_destination" ]; then
    mkdir -p "$dossier_destination"
fi

echo " Gestion des anciennes sauvegardes..."

# Compter les sauvegardes existantes
nombre_backups=$(ls -1 "$dossier_destination"/backup_*.tar.gz 2>/dev/null | wc -l)
echo "  Sauvegardes existantes : $nombre_backups"

if [ $nombre_backups -ge $max_sauvegardes ]; then
    echo "  Suppression des anciennes sauvegardes..."
    
    # Supprimer les plus anciennes (garder max_sauvegardes - 1)
    nb_a_supprimer=$((nombre_backups - max_sauvegardes + 1))
    
    # Lister les fichiers du plus ancien au plus récent et supprimer les premiers
    ls -1t "$dossier_destination"/backup_*.tar.gz 2>/dev/null | tail -n $nb_a_supprimer | while read fichier_ancien; do
        echo "    Suppression : $(basename "$fichier_ancien")"
        rm -f "$fichier_ancien"
    done
else
    echo "  Pas de nettoyage nécessaire"
fi

echo ""
```

**Modifier la création de sauvegarde :**
```bash
# === Création de la sauvegarde ===
echo " Création de la sauvegarde..."
echo "   Archivage en cours..."

# Créer l'archive avec toutes les sources valides
if tar -czf "$dossier_destination/$nom_sauvegarde" $sources_valides 2>/dev/null; then
    taille_fichier=$(du -h "$dossier_destination/$nom_sauvegarde" | cut -f1)
    echo "  Sauvegarde créée avec succès !"
    echo "  Fichier : $nom_sauvegarde"
    echo "  Taille : $taille_fichier"
else
    echo "  ERREUR lors de la création !"
    exit 1
fi

# Afficher la liste des sauvegardes actuelles
echo ""
echo "Sauvegardes disponibles :"
ls -lah "$dossier_destination"/backup_*.tar.gz 2>/dev/null | while read ligne; do
    echo "  $ligne"
done

echo ""
echo "🎉 Script terminé avec succès !"
```

**Tester :**
```bash
# Créer quelques fichiers de test supplémentaires
echo "Autre fichier" > test-data/autre.txt

# Tester plusieurs fois pour voir la rotation
./backup.sh
./backup.sh
./backup.sh
./backup.sh  # Cette fois, devrait supprimer les anciennes

# Vérifier le contenu
ls -la backups/
```

### **Point de contrôle Étape 3**
- Boucle for pour traiter plusieurs sources
- Rotation automatique des sauvegardes
- Gestion de listes et compteurs

---

## **ÉTAPE 4 : Organisation en fonctions (10 min)**

### **Objectif : Structurer le code avec des fonctions réutilisables**

### **4.1 - Découpage en fonctions**

**Créer une nouvelle version organisée :**
```bash
nano backup_functions.sh
```

**Version complète avec fonctions :**
```bash
#!/bin/bash
#
# Script de sauvegarde DataFlow - Étape 3
#

# === CONFIGURATION GLOBALE ===
nom_entreprise="DataFlow"
version_script="3.0"
dossiers_source="test-data $HOME/.bashrc"
dossier_destination="backups"
max_sauvegardes=3

# === FONCTIONS ===

# Fonction d'affichage de l'en-tête
afficher_entete() {
    echo "=== Script de sauvegarde DataFlow ==="
    echo "Entreprise : $nom_entreprise"
    echo "Version : $version_script"
    echo "Date : $(date +"%Y-%m-%d %H:%M:%S")"
    echo "Utilisateur : $USER"
    echo ""
}

# Fonction de vérification des sources
verifier_sources() {
    echo "Vérification des sources..."
    
    sources_valides=""
    total_fichiers=0
    
    for source in $dossiers_source; do
        echo "  📁 Test de : $source"
        
        if [ -e "$source" ]; then
            echo "    Trouvé"
            sources_valides="$sources_valides $source"
            
            if [ -f "$source" ]; then
                fichiers_source=1
            else
                fichiers_source=$(find "$source" -type f 2>/dev/null | wc -l)
            fi
            
            echo "    Fichiers : $fichiers_source"
            total_fichiers=$((total_fichiers + fichiers_source))
        else
            echo "    Introuvable (ignoré)"
        fi
    done
    
    echo ""
    echo "Sources valides :$sources_valides"
    echo "Total fichiers : $total_fichiers"
    
    if [ $total_fichiers -eq 0 ]; then
        echo "Aucun fichier à sauvegarder !"
        return 1  # Code d'erreur
    fi
    
    return 0  # Succès
}

# Fonction de vérification de l'espace disque
verifier_espace_disque() {
    echo "Vérification de l'espace disque..."
    
    espace_libre_kb=$(df . | awk 'NR==2 {print $4}')
    espace_libre_mb=$((espace_libre_kb / 1024))
    
    echo "Espace libre : ${espace_libre_mb} MB"
    
    if [ $espace_libre_mb -lt 100 ]; then
        echo "Peu d'espace libre"
    else
        echo "Espace suffisant"
    fi
    echo ""
}

# Fonction de rotation des sauvegardes
rotation_sauvegardes() {
    echo "Gestion des anciennes sauvegardes..."
    
    if [ ! -d "$dossier_destination" ]; then
        mkdir -p "$dossier_destination"
    fi
    
    nombre_backups=$(ls -1 "$dossier_destination"/backup_*.tar.gz 2>/dev/null | wc -l)
    echo "Sauvegardes existantes : $nombre_backups"
    
    if [ $nombre_backups -ge $max_sauvegardes ]; then
        echo "Suppression des anciennes..."
        
        nb_a_supprimer=$((nombre_backups - max_sauvegardes + 1))
        
        ls -1t "$dossier_destination"/backup_*.tar.gz 2>/dev/null | tail -n $nb_a_supprimer | while read fichier_ancien; do
            echo "  $(basename "$fichier_ancien")"
            rm -f "$fichier_ancien"
        done
    else
        echo "Pas de nettoyage nécessaire"
    fi
    echo ""
}

# Fonction de création de sauvegarde
creer_sauvegarde() {
    local nom_fichier="backup_$(date +%Y%m%d_%H%M%S).tar.gz"
    
    echo " Création de la sauvegarde..."
    echo " Fichier : $nom_fichier"
    
    if tar -czf "$dossier_destination/$nom_fichier" $sources_valides 2>/dev/null; then
        local taille=$(du -h "$dossier_destination/$nom_fichier" | cut -f1)
        echo " Sauvegarde créée !"
        echo " Taille : $taille"
        return 0
    else
        echo " ERREUR lors de la création !"
        return 1
    fi
}

# Fonction d'affichage du résumé final
afficher_resume() {
    echo ""
    echo " Sauvegardes disponibles :"
    
    if ls "$dossier_destination"/backup_*.tar.gz >/dev/null 2>&1; then
        ls -lah "$dossier_destination"/backup_*.tar.gz | while read ligne; do
            echo "  $ligne"
        done
    else
        echo "  Aucune sauvegarde trouvée"
    fi
    
    echo ""
    echo " Script terminé avec succès !"
}

# === PROGRAMME PRINCIPAL ===

# Appel des fonctions dans l'ordre logique
afficher_entete

if verifier_sources; then
    verifier_espace_disque
    rotation_sauvegardes
    
    if creer_sauvegarde; then
        afficher_resume
    else
        echo " Échec de la sauvegarde"
        exit 1
    fi
else
    echo " Échec des vérifications"
    exit 1
fi
```

**Tester :**
```bash
chmod +x backup_functions.sh
./backup_functions.sh
```

### **Point de contrôle Étape 4**
- Code organisé en fonctions logiques
- Variables locales et globales
- Codes de retour des fonctions
- Structure claire et réutilisable

---

##  **ÉTAPE 5 : Manipulation de texte et logs (10 min)**

### **Objectif : Utiliser grep, sed, awk pour analyser et filtrer**

### **5.1 - Ajout d'un système de logs et d'analyse**

**Ajouter au début du script (après la configuration) :**
```bash
# === CONFIGURATION LOGS ===
fichier_log="dataflow_backup.log"

# Fonction de logging
ecrire_log() {
    local niveau="$1"
    local message="$2"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$niveau] $message" >> "$fichier_log"
}
```

**Modifier la fonction de création de sauvegarde pour ajouter les logs :**
```bash
# Fonction de création de sauvegarde avec logs
creer_sauvegarde() {
    local nom_fichier="backup_$(date +%Y%m%d_%H%M%S).tar.gz"
    
    echo " Création de la sauvegarde..."
    echo " Fichier : $nom_fichier"
    
    ecrire_log "INFO" "Début création sauvegarde: $nom_fichier"
    ecrire_log "INFO" "Sources: $sources_valides"
    
    if tar -czf "$dossier_destination/$nom_fichier" $sources_valides 2>/dev/null; then
        local taille=$(du -h "$dossier_destination/$nom_fichier" | cut -f1)
        echo " Sauvegarde créée !"
        echo " Taille : $taille"
        
        ecrire_log "SUCCESS" "Sauvegarde créée: $nom_fichier (Taille: $taille)"
        return 0
    else
        echo " ERREUR lors de la création !"
        ecrire_log "ERROR" "Échec création sauvegarde: $nom_fichier"
        return 1
    fi
}
```

**Ajouter une fonction d'analyse des logs à la fin :**
```bash
# Fonction d'analyse des logs avec grep, sed, awk
analyser_logs() {
    echo "Analyse des logs..."
    
    if [ ! -f "$fichier_log" ]; then
        echo "  Aucun log à analyser"
        return
    fi
    
    # Utilisation de grep pour filtrer
    echo "   Dernières activités (grep) :"
    grep "$(date +%Y-%m-%d)" "$fichier_log" | tail -5 | while read ligne; do
        echo "    $ligne"
    done
    
    echo ""
    
    # Utilisation de awk pour compter
    echo "   Statistiques (awk) :"
    total_lignes=$(wc -l < "$fichier_log")
    echo "    Total événements : $total_lignes"
    
    succes_count=$(grep -c "SUCCESS" "$fichier_log" 2>/dev/null || echo 0)
    error_count=$(grep -c "ERROR" "$fichier_log" 2>/dev/null || echo 0)
    
    echo "    Succès : $succes_count"
    echo "    Erreurs : $error_count"
    
    # Utilisation de sed pour formater
    echo ""
    echo "   Dernières sauvegardes réussies (sed + grep) :"
    grep "Sauvegarde créée" "$fichier_log" | tail -3 | sed 's/.*SUCCESS]//' | while read ligne; do
        echo "    $ligne"
    done
    
    echo ""
}
```

**Ajouter l'appel de la fonction d'analyse avant la fin du programme principal :**
```bash
# À la fin du programme principal, avant afficher_resume
    if creer_sauvegarde; then
        analyser_logs        # NOUVELLE LIGNE
        afficher_resume
    else
```

**Version finale complète - créer le script final :**
```bash
nano backup_final.sh
```

**Copier tout le contenu de backup_functions.sh et ajouter les modifications ci-dessus**

**Tester le script final :**
```bash
chmod +x backup_final.sh

# Tester plusieurs fois pour générer des logs
./backup_final.sh
./backup_final.sh
./backup_final.sh

# Examiner le fichier de log créé
cat dataflow_backup.log

# Tester manuellement les outils de manipulation de texte
echo "🧪 Tests manuels :"
echo "Logs d'aujourd'hui :"
grep "$(date +%Y-%m-%d)" dataflow_backup.log

echo ""
echo "Nombre de succès :"
grep -c "SUCCESS" dataflow_backup.log

echo ""
echo "Formatage avec sed :"
grep "SUCCESS" dataflow_backup.log | sed 's/.*]//'
```

### ** Point de contrôle Étape 5**
- Système de logs intégré
- Utilisation de grep pour filtrer
- Utilisation de awk pour compter
- Utilisation de sed pour formater
- Analyse automatique des données

---

##  **LIVRABLE FINAL**

### **Créer un rapport de l'atelier**

```bash
nano rapport_atelier_[VOTRE_NOM].txt
```

**Contenu du rapport :**
```
=== RAPPORT ATELIER SCRIPTS SHELL DATAFLOW ===
Nom : [Votre nom]
Date : [Date de l'atelier]

=== SCRIPT DÉVELOPPÉ ===
Nom du script final : backup_final.sh
Fonctionnalité : Sauvegarde intelligente avec rotation et analyse

=== CONCEPTS BASH MAÎTRISÉS ===
□ Variables et substitution de commandes
□ Conditions et tests de fichiers
□ Boucles for et while
□ Fonctions avec paramètres et codes de retour
□ Manipulation de texte (grep, sed, awk)

=== TESTS DE VALIDATION ===
Copier-coller le résultat de ces commandes :

$ ./backup_final.sh

$ ls -la backups/

$ cat dataflow_backup.log

$ grep "SUCCESS" dataflow_backup.log | wc -l

=== STRUCTURE FINALE CRÉÉE ===
$ find . -name "*.sh" -o -name "*.log" -o -name "*.tar.gz" | head -10

=== ANALYSE PERSONNELLE ===
1. Concept le plus difficile :
2. Concept le plus utile :
3. Amélioration possible du script :
```

### **Tests de validation finale**

```bash
# Test 1 : Script fonctionne-t-il ?
./backup_final.sh

# Test 2 : Logs sont-ils créés ?
[ -f dataflow_backup.log ] && echo " Logs OK" || echo " Pas de logs"

# Test 3 : Sauvegardes sont-elles créées ?
[ -d backups ] && echo " Dossier backup OK" || echo " Pas de dossier backup"

# Test 4 : Rotation fonctionne-t-elle ?
./backup_final.sh
./backup_final.sh
./backup_final.sh
./backup_final.sh
count=$(ls backups/*.tar.gz 2>/dev/null | wc -l)
echo "Nombre de sauvegardes : $count (doit être ≤ 3)"

# Test 5 : Analyse des logs fonctionne-t-elle ?
grep "" dataflow_backup.log >/dev/null && echo " Analyse OK" || echo " Pas d'analyse"
```

---

##  **CHECKLIST D'AUTO-VALIDATION**

**Concepts Bash maîtrisés :**
- [ ] Variables (déclaration, utilisation, substitution)
- [ ] Conditions (if/then/else, tests de fichiers, tests numériques)
- [ ] Boucles (for avec listes, compteurs)
- [ ] Fonctions (définition, appel, codes de retour)
- [ ] Manipulation texte (grep, sed, awk de base)

**Script fonctionnel :**
- [ ] Sauvegarde plusieurs sources
- [ ] Vérifie les prérequis
- [ ] Fait de la rotation automatique
- [ ] Génère des logs
- [ ] Analyse ses propres données

**Bonnes pratiques :**
- [ ] Code organisé en fonctions
- [ ] Variables configurables en haut
- [ ] Gestion d'erreurs avec exit
- [ ] Messages informatifs pour l'utilisateur
- [ ] Logs pour traçabilité
