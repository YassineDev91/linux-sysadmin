# Atelier pratique - Séance 6 : Virtualisation vs Conteneurisation
## Durée : 1h15

---

## **SCÉNARIO PROFESSIONNEL**

Vous êtes stagiaire chez **CloudTech Solutions**, une entreprise qui conseille ses clients sur les technologies modernes. Votre mission : découvrir et comparer deux approches pour faire tourner des applications : les **machines virtuelles classiques** et les **conteneurs Docker**. Vous allez créer la même application simple dans les deux environnements et observer les différences !

**Objectif :** Comprendre concrètement quand utiliser des VMs ou des conteneurs selon les besoins.

---

## **PRÉREQUIS ET MATÉRIEL**

### **Prérequis obligatoires**
- [ ] **Machine avec 8GB RAM minimum** (6GB = limite absolue)
- [ ] **15GB espace disque libre** minimum
- [ ] **Connexion Internet stable** (téléchargements)
- [ ] **Droits administrateur** sur votre machine
- [ ] **VirtualBox installé** et testé (voir guide ci-dessous)

### **Systèmes supportés**
- **Windows 10/11** : Docker Desktop + VirtualBox
- **macOS** : Docker Desktop + VirtualBox  
- **Linux Ubuntu** : Docker + VirtualBox (plus simple)

### ** Vérification VirtualBox**
```bash
# Tester que VirtualBox fonctionne
# 1. Ouvrir VirtualBox
# 2. Créer une VM test (1GB RAM, pas d'installation)
# 3. Si ça fonctionne → OK, sinon demander de l'aide
```

---

## **PLANNING DÉTAILLÉ**

- **00-15 min** : Vérification setup et préparation
- **15-40 min** : Création VM avec application simple  
- **40-65 min** : Installation Docker et conteneur
- **65-75 min** : Comparaison et observations

** Points de contrôle toutes les 15 minutes !**

---

## **PARTIE 1 : Machine Virtuelle Ubuntu**

### **Objectif : Créer une VM avec une application web simple**

### **1.1 - Création VM rapide**

**Configuration simplifiée :**
```
1. Ouvrir VirtualBox → "Nouvelle"
2. Nom : CloudTech-VM
3. Type : Linux, Ubuntu (64-bit)
4. RAM : 1024 MB (1GB suffit pour notre test)
5. Disque : 8GB dynamique
6. Réseau : NAT (par défaut)
7. Ne pas encore démarrer !
```

**POINT CONTRÔLE 1: VM créée mais pas démarrée**

### **1.2 - Installation Ubuntu Server minimale**

**Version ultra-rapide :**
```bash
# 1. Télécharger Ubuntu Server 22.04 LTS (pendant que les autres finissent)
# 2. Monter l'ISO dans la VM
# 3. Démarrer l'installation EXPRESS :
   - Langue : English (plus rapide)
   - Installation : Minimale
   - Nom utilisateur : cloudtech
   - Mot de passe : cloudtech123
   - SSH : OUI
   - Skip autres options
```

**Parallèle pendant l'installation :** Les étudiants rapides peuvent commencer à préparer Docker

### **1.3 - Application web ultra-simple**

**Une fois Ubuntu installé :**
```bash
# Se connecter à la VM
sudo apt update

# Créer une page web statique simple
mkdir webapp
cd webapp

# Créer index.html (TRÈS SIMPLE)
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>CloudTech VM</title></head>
<body style="text-align: center; font-family: Arial; padding: 50px;">
    <h1>🖥️ Application VM</h1>
    <p>Cette page tourne dans une <strong>Machine Virtuelle</strong></p>
    <p>Serveur : VM Ubuntu</p>
    <p>Technologie : Virtualisation classique</p>
    <button onclick="location.reload()">Actualiser</button>
    <hr>
    <small>CloudTech Solutions - Atelier Virtualisation</small>
</body>
</html>
EOF

# Démarrer le serveur web (TRÈS SIMPLE)
python3 -m http.server 8080 &

# Tester localement
curl http://localhost:8080
```

**Vérification :** Page web accessible dans la VM

**POINT CONTRÔLE 2: VM avec application web fonctionnelle**

---

## **PARTIE 2 : Conteneur Docker**

### **Objectif : Créer la même application dans un conteneur**

### **2.1 - Installation Docker simplifiée**

**Sur Windows/macOS :**
```bash
# 1. Télécharger Docker Desktop depuis docker.com
# 2. Installer (redémarrage possible)
# 3. Démarrer Docker Desktop
# 4. Tester : docker --version
```

**Sur Linux Ubuntu :**
```bash
# Installation automatique
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Se reconnecter ou utiliser sudo
docker --version
docker run hello-world
```

**⚠️ Problèmes courants et solutions :**
- **Windows :** Activer la virtualisation dans le BIOS
- **macOS :** Autoriser Docker dans les préférences système
- **Linux :** Si erreur permissions, utiliser `sudo docker`

### **2.2 - Création conteneur simple**

**Version ultra-simplifiée :**
```bash
# Créer dossier pour Docker
mkdir docker-webapp
cd docker-webapp

# Créer la même page HTML
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>CloudTech Docker</title></head>
<body style="text-align: center; font-family: Arial; padding: 50px;">
    <h1>Application Docker</h1>
    <p>Cette page tourne dans un <strong>Conteneur Docker</strong></p>
    <p>Serveur : Container Linux</p>
    <p>Technologie : Conteneurisation</p>
    <button onclick="location.reload()">Actualiser</button>
    <hr>
    <small>CloudTech Solutions - Atelier Virtualisation</small>
</body>
</html>
EOF

# Dockerfile TRÈS SIMPLE
cat > Dockerfile << 'EOF'
# Utiliser une image avec serveur web intégré
FROM nginx:alpine

# Copier notre page
COPY index.html /usr/share/nginx/html/

# Le port 80 est déjà exposé par nginx
EOF
```

### **2.3 - Construction et lancement**

```bash
# Construire l'image
docker build -t cloudtech-app .

# Lancer le conteneur
docker run -d -p 8081:80 --name cloudtech-container cloudtech-app

# Vérifier que ça marche
docker ps
curl http://localhost:8081
```

**POINT CONTRÔLE 3 : Docker fonctionnel avec application**

---

## **PARTIE 3 : Comparaison côte à côte**

### **Objectif : Observer les différences concrètes**

### **3.1 - Tests simples de performance**

**Script de test automatique :**
```bash
# Créer un script de comparaison simple
cat > comparaison.sh << 'EOF'
#!/bin/bash

echo "=== COMPARAISON VM vs DOCKER ==="
echo "Date : $(date)"
echo ""

echo "🔍 Test 1 : Accès aux applications"
echo "VM (port 8080) :"
curl -s http://localhost:8080 | grep -o '<h1>.*</h1>' || echo "❌ Non accessible"

echo "Docker (port 8081) :"
curl -s http://localhost:8081 | grep -o '<h1>.*</h1>' || echo "❌ Non accessible"

echo ""
echo "🔍 Test 2 : Temps de réponse (approximatif)"
echo "VM :"
time curl -s http://localhost:8080 > /dev/null 2>&1 || echo "Non testé"

echo "Docker :"
time curl -s http://localhost:8081 > /dev/null 2>&1 || echo "Non testé"

echo ""
echo "🔍 Test 3 : Utilisation ressources"
echo "Processus VM (VirtualBox) :"
ps aux | grep -i virtualbox | grep -v grep | wc -l

echo "Processus Docker :"
docker ps --format "table {{.Names}}\t{{.Status}}"

echo ""
echo "✅ Tests terminés !"
EOF

chmod +x comparaison.sh
./comparaison.sh
```

### **3.2 - Observations guidées**

**Tableau à remplir en direct :**
```
| Critère | Machine Virtuelle | Conteneur Docker | Vos observations |
|---------|-------------------|------------------|------------------|
| Facilité setup | ⭐⭐⭐ (long) | ⭐⭐⭐⭐ (rapide) | |
| Temps démarrage | ~2 minutes | ~2 secondes | |
| RAM utilisée | 1024 MB allouée | ~20 MB réels | |
| Espace disque | ~8 GB | ~50 MB | |
| Isolation | Complète (OS séparé) | Processus seulement | |
```

**Questions de réflexion :**
1. Quelle solution a été plus facile à mettre en place ?
2. Laquelle utilise le moins de ressources ?
3. Pour quel type de projet utiliseriez-vous chaque technologie ?

**POINT CONTRÔLE FINAL: Comparaison terminée**

---

## **LIVRABLE SIMPLIFIÉ**

### **Créer : `rapport_virtualisation_[VOTRE_NOM].txt`**

```
=== RAPPORT ATELIER VIRTUALISATION CLOUDTECH ===
Nom : [Votre nom]
Date : [Date]

=== MES RÉUSSITES ===
□ VM Ubuntu créée et fonctionnelle
□ Application web dans la VM accessible  
□ Docker installé sur ma machine
□ Conteneur avec application lancé
□ Comparaison des deux approches

=== MES OBSERVATIONS ===

1. FACILITÉ D'UTILISATION
Plus facile à installer : VM / Docker (barrer l'inutile)
Pourquoi : ________________________________

2. PERFORMANCES
Plus rapide au démarrage : VM / Docker  
Utilise moins de RAM : VM / Docker
Utilise moins d'espace disque : VM / Docker

3. RESSOURCES CONSOMMÉES
RAM machine pendant l'atelier :
- Avant l'atelier : _____ MB utilisés
- Avec la VM seule : _____ MB utilisés  
- Avec VM + Docker : _____ MB utilisés

4. APPLICATIONS ACCESSIBLES
□ http://localhost:8080 (VM) → Fonctionne / Ne fonctionne pas
□ http://localhost:8081 (Docker) → Fonctionne / Ne fonctionne pas

=== MA RECOMMANDATION ===

Pour quels projets utiliser les VMs :
1. ________________________________
2. ________________________________

Pour quels projets utiliser Docker :
1. ________________________________  
2. ________________________________

=== DIFFICULTÉS RENCONTRÉES ===
Problème principal : ________________________________
Solution trouvée : ________________________________

=== COMMANDES IMPORTANTES RETENUES ===
VirtualBox : ________________________________
Docker : ________________________________

=== SCORE D'AUTO-ÉVALUATION ===
Ma compréhension VM vs Docker : ___/10
Ma capacité à choisir la bonne techno : ___/10
```

---

## **CHECKLIST SIMPLIFIÉE**

### **Réalisations techniques :**
- [ ] **VM créée** avec Ubuntu et application web
- [ ] **Docker installé** et premier conteneur fonctionnel  
- [ ] **Applications accessibles** sur les deux ports
- [ ] **Tests de comparaison** effectués

### **Compréhension :**
- [ ] **Différence conceptuelle** VM vs conteneurs comprise
- [ ] **Avantages/inconvénients** de chaque approche identifiés
- [ ] **Cas d'usage** appropriés pour chaque technologie
- [ ] **Impact ressources** observé concrètement

### **Compétences acquises :**
- [ ] **Création de VM** basique avec VirtualBox
- [ ] **Commandes Docker** de base maîtrisées
- [ ] **Comparaison technique** argumentée
- [ ] **Recommandations** justifiées selon le contexte

**Score minimum pour valider l'atelier : 8/12 éléments cochés**

---

## **GUIDE DE DÉPANNAGE RAPIDE**

### **Problèmes VM fréquents :**
```
❌ VM ne démarre pas → Vérifier virtualisation activée dans BIOS
❌ Installation Ubuntu lente → Passer en anglais, installation minimale
❌ Pas d'accès web → Vérifier que python3 -m http.server tourne
❌ VM très lente → Allouer plus de RAM (2GB au lieu de 1GB)
```

### **Problèmes Docker fréquents :**
```
❌ Docker ne s'installe pas → Vérifier droits admin + redémarrer
❌ "docker: command not found" → Redémarrer terminal / se reconnecter
❌ Port 8081 déjà utilisé → Utiliser -p 8082:80 à la place
❌ Image ne se construit pas → Vérifier Dockerfile et être dans le bon dossier
```


