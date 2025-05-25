# Atelier Pratique : Configuration réseau sous Linux
## Durée : 1 heure
## Niveau : Bac+3 débutant

---

## **SCÉNARIO PROFESSIONNEL**

Vous êtes stagiaire administrateur système chez **TechCorp**, une startup qui développe des applications web. Votre responsable vous confie la configuration d'un nouveau poste de travail Ubuntu qui servira de station d'administration. Cette machine doit avoir une adresse IP statique, être sécurisée avec un pare-feu, et permettre l'accès SSH sécurisé. Votre mission : appliquer les bonnes pratiques vues en cours pour transformer ce poste en station d'administration professionnelle.

---

## **MATÉRIEL NÉCESSAIRE**

- **1 machine virtuelle** : Ubuntu Desktop/Server 22.04 LTS
- **Accès administrateur** : Compte utilisateur avec droits sudo
- **Connexion réseau** : Interface réseau active
- **Terminal** : Accès à la ligne de commande

---

## **ÉTAPE 1 : Diagnostic et configuration IP statique (15 min)**

### **Objectif**
Analyser la configuration réseau actuelle et configurer une adresse IP statique selon les standards de l'entreprise.

### **1.1 - Diagnostic avec les commandes de base**

**Ce que nous faisons :** Utiliser les 3 commandes de base vues en cours pour comprendre l'état actuel du réseau.

```bash
# Les 3 commandes essentielles du cours
ip addr show        # Voir mes adresses IP
ip route show       # Voir mes chemins réseau  
ip link show        # Voir mes interfaces réseau
```

**Identification de l'interface :** Notez le nom de votre interface réseau principale (ex: ens33, enp0s3). Vous la reconnaissez car elle a une adresse IP autre que 127.0.0.1.

### **1.2 - Tests de connectivité selon la méthode 4 étapes**

**Ce que nous faisons :** Appliquer la méthode de diagnostic systématique vue en cours.

```bash
# Méthode en 4 étapes du cours
ping -c 2 127.0.0.1        # Étape 1 : Ma machine fonctionne ?
ping -c 2 192.168.1.1      # Étape 2 : Mon réseau local fonctionne ?
ping -c 2 8.8.8.8          # Étape 3 : Internet fonctionne ?
ping -c 2 google.com       # Étape 4 : DNS fonctionne ?
```

**Important :** Remplacez 192.168.1.1 par l'adresse de votre passerelle affichée dans `ip route show`.

### **1.3 - Configuration IP manuelle (temporaire)**

**Ce que nous faisons :** Appliquer la configuration manuelle vue en cours, adaptée aux standards TechCorp.

```bash
# Configuration manuelle selon le cours
# Remplacez ens33 par votre interface
sudo ip addr add 192.168.100.50/24 dev ens33
sudo ip link set ens33 up
sudo ip route add default via 192.168.100.1
```

**Vérification immédiate :**
```bash
ip addr show ens33
ping -c 3 192.168.100.1
```

### **1.4 - Configuration permanente avec Netplan**

**Ce que nous faisons :** Rendre la configuration persistante en utilisant Netplan comme vu en cours.

```bash
# Créer le fichier de configuration Netplan
sudo nano /etc/netplan/01-techcorp.yaml
```

**Contenu du fichier (format exact du cours) :**
```yaml
network:
  version: 2
  ethernets:
    ens33:  # Adaptez selon votre interface
      addresses: [192.168.100.50/24]
      routes:
        - to: default
          via: 192.168.100.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
# Appliquer la configuration (commande du cours)
sudo netplan apply
```

**Test final :** Répéter la méthode 4 étapes pour confirmer que tout fonctionne.

### **Résultat attendu**
- Configuration IP statique active
- Connectivité Internet confirmée avec la méthode 4 étapes
- Configuration persistante après redémarrage

---

## **ÉTAPE 2 : Configuration DNS et résolution de noms (10 min)**

### **Objectif**
Configurer la résolution de noms selon les pratiques vues en cours.

### **2.1 - Configuration du fichier /etc/hosts**

**Ce que nous faisons :** Ajouter des entrées locales comme montré en cours.

```bash
# Modifier le fichier hosts (méthode du cours)
sudo nano /etc/hosts
```

**Ajouter ces lignes selon le format du cours :**
```
# Format du cours : IP    nom
127.0.0.1       localhost
192.168.100.50  admin-station
192.168.100.10  serveur-web
192.168.100.20  serveur-db
```

### **2.2 - Tests de résolution DNS**

**Ce que nous faisons :** Tester la résolution comme expliqué en cours.

```bash
# Tests selon le cours
ping admin-station     # Utilise /etc/hosts
ping google.com        # Utilise DNS
```

**Vérification de la configuration DNS :**
```bash
cat /etc/resolv.conf   # Voir les serveurs DNS configurés
```

### **Résultat attendu**
- Résolution locale fonctionnelle via /etc/hosts
- Résolution Internet via DNS opérationnelle

---

## **ÉTAPE 3 : Configuration du pare-feu avec UFW (15 min)**

### **Objectif**
Sécuriser la station avec UFW selon les méthodes vues en cours.

### **3.1 - Installation et configuration de base**

**Ce que nous faisons :** Suivre les étapes d'installation et de configuration de base du cours.

```bash
# Installation et activation (procédure du cours)
sudo apt install ufw
sudo ufw enable
```

### **3.2 - Règles essentielles pour une station d'administration**

**Ce que nous faisons :** Appliquer les règles vues en cours, adaptées au contexte.

```bash
# Règles de base du cours adaptées
sudo ufw allow 22/tcp      # SSH (administration)
sudo ufw allow 80/tcp      # Sites web HTTP  
sudo ufw allow 443/tcp     # Sites web HTTPS
```

**Voir les règles configurées :**
```bash
sudo ufw status
```

### **3.3 - Règles avancées du cours**

**Ce que nous faisons :** Appliquer quelques règles avancées vues en cours.

```bash
# Exemple du cours : autoriser SSH depuis le réseau local seulement
sudo ufw allow from 192.168.100.0/24 to any port 22

# Supprimer la règle SSH générale (méthode du cours)
sudo ufw delete allow 22/tcp
```

### **Résultat attendu**
- Pare-feu UFW actif
- Règles de sécurité appropriées configurées
- SSH accessible uniquement depuis le réseau local

---

## **ÉTAPE 4 : Configuration SSH sécurisée (15 min)**

### **Objectif**
Configurer SSH avec authentification par clés selon les bonnes pratiques du cours.

### **4.1 - Génération des clés SSH**

**Ce que nous faisons :** Générer des clés SSH avec la méthode exacte du cours.

```bash
# Génération de clés (commande exacte du cours)
ssh-keygen -t ed25519
```

**Important :** Appuyez sur Entrée pour toutes les questions comme indiqué dans le cours.

### **4.2 - Test de connexion locale**

**Ce que nous faisons :** Tester la configuration SSH localement.

```bash
# Copier la clé pour test local (adaptation du cours)
ssh-copy-id $USER@localhost

# Test de connexion (format du cours)
ssh $USER@localhost
```

### **4.3 - Sécurisation du serveur SSH**

**Ce que nous faisons :** Appliquer la sécurisation optionnelle mentionnée dans le cours.

```bash
# Modifier la configuration SSH (méthode du cours)
sudo nano /etc/ssh/sshd_config
```

**Ajouter/modifier cette ligne (sécurisation du cours) :**
```
PasswordAuthentication no
```

```bash
# Redémarrer SSH pour appliquer les changements
sudo systemctl restart ssh
```

### **4.4 - Validation de la configuration**

**Ce que nous faisons :** Vérifier que SSH fonctionne avec les clés.

```bash
# Test de connexion sans mot de passe
ssh $USER@admin-station
```

### **Résultat attendu**
- Clés SSH générées et configurées
- Connexion SSH sans mot de passe fonctionnelle
- Authentification par mot de passe désactivée

---

## **ÉTAPE 5 : Diagnostic final et validation (5 min)**

### **Objectif**
Valider l'ensemble de la configuration avec les outils du cours.

### **5.1 - Script de diagnostic automatique du cours**

**Ce que nous faisons :** Utiliser le script de diagnostic vu en cours.

```bash
# Créer le script de diagnostic du cours
nano /tmp/diagnostic.sh
```

**Contenu exact du script du cours :**
```bash
#!/bin/bash
echo "=== DIAGNOSTIC RÉSEAU ==="

echo "1. Test machine locale :"
ping -c 1 127.0.0.1 >/dev/null && echo "✅ OK" || echo "❌ ERREUR"

echo "2. Test passerelle :"
GATEWAY=$(ip route | grep default | awk '{print $3}')
ping -c 1 $GATEWAY >/dev/null && echo "✅ OK" || echo "❌ ERREUR"

echo "3. Test Internet :"
ping -c 1 8.8.8.8 >/dev/null && echo "✅ OK" || echo "❌ ERREUR"

echo "4. Test DNS :"
ping -c 1 google.com >/dev/null && echo "✅ OK" || echo "❌ ERREUR"

echo "=== FIN DIAGNOSTIC ==="
```

```bash
# Exécuter le script
chmod +x /tmp/diagnostic.sh
/tmp/diagnostic.sh
```

### **5.2 - Récapitulatif avec les commandes essentielles du cours**

**Ce que nous faisons :** Vérifier la configuration finale avec les commandes du cours.

```bash
# Commandes essentielles du cours pour validation
ip addr show                    # Voir mes IP
ip route show                   # Voir mes routes
sudo ufw status                 # Voir les règles pare-feu
ssh $USER@admin-station 'hostname'  # Test SSH
```

### **Résultat attendu**
- Tous les tests du script de diagnostic passent
- Configuration réseau statique opérationnelle
- Sécurité pare-feu et SSH configurée

---

## **LIVRABLE : RAPPORT DE CONFIGURATION TECHCORP**

Créez un fichier `rapport_config_techcorp_[VOTRE_NOM].txt` contenant :

### **1. Configuration réseau appliquée**
```
Adresse IP statique : _______________
Passerelle : _______________
DNS configurés : _______________

Méthode 4 étapes - Résultats :
- Test local (127.0.0.1) : ✓/✗
- Test passerelle : ✓/✗  
- Test Internet (8.8.8.8) : ✓/✗
- Test DNS (google.com) : ✓/✗
```

### **2. Configuration DNS locale**
```
Entrées ajoutées dans /etc/hosts :
(Copier le contenu de votre fichier /etc/hosts)

Test résolution locale : ✓/✗
```

### **3. Configuration pare-feu UFW**
```
(Copier la sortie de : sudo ufw status)

Règles configurées : ✓/✗
SSH sécurisé (réseau local uniquement) : ✓/✗
```

### **4. Configuration SSH**
```
Clés SSH générées : ✓/✗
Connexion sans mot de passe : ✓/✗
Authentification par mot de passe désactivée : ✓/✗
```

### **5. Script de diagnostic**
```
(Copier la sortie du script de diagnostic du cours)

Tous les tests passent : ✓/✗
```

### **6. Difficultés rencontrées**
Décrivez les problèmes rencontrés et comment vous les avez résolus.

---

## **CHECKLIST D'AUTO-VALIDATION**

Cochez chaque élément validé :

**Configuration réseau :**
- [ ] IP statique configurée avec les commandes du cours
- [ ] Configuration Netplan persistante créée
- [ ] Méthode 4 étapes du cours validée
- [ ] Tests de connectivité réussis

**Résolution de noms :**
- [ ] Fichier /etc/hosts configuré selon le cours
- [ ] Tests de résolution locale réussis
- [ ] Configuration DNS fonctionnelle

**Sécurité pare-feu :**
- [ ] UFW installé et activé selon le cours
- [ ] Règles essentielles configurées
- [ ] SSH limité au réseau local
- [ ] Règles avancées du cours appliquées

**SSH sécurisé :**
- [ ] Clés SSH générées avec ed25519
- [ ] Connexion sans mot de passe fonctionnelle
- [ ] Authentification par mot de passe désactivée
- [ ] Configuration serveur SSH sécurisée

**Validation finale :**
- [ ] Script de diagnostic du cours exécuté avec succès
- [ ] Toutes les commandes essentielles du cours validées

**Score final : ___ / 17 éléments validés**

---

## **CONSEILS POUR RÉUSSIR**

1. **Respectez exactement les commandes du cours** 
2. **Utilisez la méthode 4 étapes** pour tous vos tests
3. **Adaptez les noms d'interfaces** selon votre configuration
4. **Testez après chaque étape** avant de continuer
5. **Utilisez le script de diagnostic** régulièrement

**Temps recommandé :**
- Étape 1 : 15 minutes (diagnostic et IP statique)
- Étape 2 : 10 minutes (DNS)
- Étape 3 : 15 minutes (pare-feu)  
- Étape 4 : 15 minutes (SSH)
- Étape 5 : 5 minutes (validation)

**Bonne chance ! 🚀**