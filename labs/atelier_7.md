# Atelier Pratique Docker - Introduction Complète

**Durée :** 1 heure  
**Objectif :** Maîtriser les concepts fondamentaux de Docker  
**Livrable :** Image Docker personnalisée + rapport de validation

---

## **Objectif de l'atelier**

Créer et déployer une **application web simple** en appliquant toutes les notions vues en cours :
- Manipulation des conteneurs
- Création d'images avec Dockerfile (+4pts)
- Gestion des volumes et données
- Configuration des réseaux
- Bonnes pratiques de sécurité (+2pts)

**Livrable final :** Image Docker `mon-app:final` + fichier `validation-docker.txt` avec toutes vos vérifications.

---

## **Application à conteneuriser**

### **Code source fourni**

Créez cette structure de projet simple :

```
docker-atelier/
├── app.py
├── requirements.txt
├── templates/
│   └── index.html
└── static/
    └── style.css
```

**app.py** (Application Flask simple)
```python
from flask import Flask, render_template, request
import datetime
import os

app = Flask(__name__)

# Configuration via variables d'environnement
app.config['DEBUG'] = os.environ.get('DEBUG', 'False').lower() == 'true'
PORT = int(os.environ.get('PORT', 5000))

# Stockage simple en mémoire (sera remplacé par volume)
messages = []

@app.route('/')
def home():
    return render_template('index.html', messages=messages)

@app.route('/add', methods=['POST'])
def add_message():
    message = request.form.get('message')
    if message:
        messages.append({
            'text': message,
            'timestamp': datetime.datetime.now().strftime('%H:%M:%S')
        })
    return render_template('index.html', messages=messages)

@app.route('/health')
def health():
    return {'status': 'ok', 'time': datetime.datetime.now().isoformat()}

if __name__ == '__main__':
    print(f"🚀 App starting on port {PORT}")
    app.run(host='0.0.0.0', port=PORT, debug=app.config['DEBUG'])
```

**requirements.txt**
```
Flask==2.3.2
```

**templates/index.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Mon App Docker</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <div class="container">
        <h1>🐳 Mon Application Docker</h1>
        <p>Application conteneurisée avec succès !</p>
        
        <form method="POST" action="/add">
            <input type="text" name="message" placeholder="Tapez votre message" required>
            <button type="submit">Ajouter</button>
        </form>
        
        <div class="messages">
            {% for msg in messages %}
                <div class="message">
                    <span class="time">{{ msg.timestamp }}</span>: {{ msg.text }}
                </div>
            {% endfor %}
        </div>
    </div>
</body>
</html>
```

**static/style.css**
```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f0f8ff;
}

.container {
    max-width: 600px;
    margin: 0 auto;
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

h1 {
    color: #2196F3;
    text-align: center;
}

form {
    margin: 20px 0;
    text-align: center;
}

input[type="text"] {
    padding: 10px;
    width: 300px;
    border: 1px solid #ddd;
    border-radius: 5px;
}

button {
    padding: 10px 20px;
    background: #2196F3;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.messages {
    margin-top: 20px;
}

.message {
    padding: 10px;
    margin: 5px 0;
    background: #e3f2fd;
    border-left: 4px solid #2196F3;
}

.time {
    font-weight: bold;
    color: #666;
}
```

---

## 🐳 **Étape 1 : Manipulation des conteneurs (15 min)**

### **1.1 Découverte des images disponibles**

```bash
# Rechercher des images Python sur Docker Hub
docker search python

# Télécharger l'image Python officielle
docker pull python:3.9-slim

# Lister vos images locales
docker images
```

**VÉRIFICATION 1 :**
Copiez le résultat de `docker images` dans votre fichier `validation-docker.txt` :
```bash
echo "=== ÉTAPE 1.1 : Images disponibles ===" >> validation-docker.txt
docker images >> validation-docker.txt
echo "" >> validation-docker.txt
```

### **1.2 Premier conteneur interactif**

```bash
# Lancer Python en mode interactif
docker run -it --name mon-python python:3.9-slim python

# Dans le conteneur Python, tapez :
>>> print("Hello Docker!")
>>> import sys
>>> print(f"Python version: {sys.version}")
>>> exit()
```

**VÉRIFICATION 2 :**
```bash
# Vérifier le conteneur créé
docker ps -a

echo "=== ÉTAPE 1.2 : Premier conteneur ===" >> validation-docker.txt
docker ps -a | grep mon-python >> validation-docker.txt
echo "" >> validation-docker.txt
```

**Question de compréhension :** Pourquoi le conteneur apparaît-il avec le statut "Exited" dans `docker ps -a` ?

### **1.3 Conteneur en arrière-plan**

```bash
# Nettoyer le conteneur précédent
docker rm mon-python

# Lancer un serveur web simple
docker run -d --name web-test -p 8080:80 nginx

# Vérifier qu'il fonctionne
curl http://localhost:8080
# Ou ouvrez http://localhost:8080 dans votre navigateur
```

**VÉRIFICATION 3 :**
```bash
echo "=== ÉTAPE 1.3 : Conteneur en arrière-plan ===" >> validation-docker.txt
docker ps >> validation-docker.txt
docker logs web-test | head -5 >> validation-docker.txt
echo "" >> validation-docker.txt
```

**Question de compréhension :** Expliquez la différence entre `-it` et `-d` dans les commandes docker run.

### **1.4 Inspection et debugging**

```bash
# Inspecter la configuration du conteneur
docker inspect web-test

# Voir les processus qui tournent
docker top web-test

# Voir les statistiques en temps réel
docker stats web-test --no-stream

# Exécuter une commande dans le conteneur
docker exec -it web-test /bin/bash
# Dans le conteneur : ls /usr/share/nginx/html/
# Puis tapez : exit
```

**VÉRIFICATION 4 :**
```bash
echo "=== ÉTAPE 1.4 : Inspection conteneur ===" >> validation-docker.txt
docker inspect web-test | grep -A5 -B5 "IPAddress" >> validation-docker.txt
docker top web-test >> validation-docker.txt
echo "" >> validation-docker.txt

# Nettoyer
docker stop web-test
docker rm web-test
```

---

## 🔨 **Étape 2 : Création d'images avec Dockerfile (20 min)**

### **2.1 Préparation du projet**

```bash
# Créer la structure du projet
mkdir docker-atelier
cd docker-atelier
mkdir templates static

# Créer tous les fichiers avec le code fourni ci-dessus
# (app.py, requirements.txt, templates/index.html, static/style.css)
```

### **2.2 Dockerfile de base**

Créez le fichier `Dockerfile` :

```dockerfile
# Image de base Python légère
FROM python:3.9-slim

# Métadonnées
LABEL maintainer="votre-nom"
LABEL description="Application Flask de démonstration Docker"

# Variables d'environnement
ENV PYTHONUNBUFFERED=1
ENV FLASK_APP=app.py
ENV PORT=5000

# Répertoire de travail
WORKDIR /app

# Copier les dépendances en premier (optimisation cache)
COPY requirements.txt .

# Installer les dépendances
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code source
COPY . .

# Port exposé
EXPOSE 5000

# Commande par défaut
CMD ["python", "app.py"]
```

**VÉRIFICATION 5 :**
```bash
# Construire l'image
docker build -t mon-app:v1 .

echo "=== ÉTAPE 2.2 : Première image ===" >> validation-docker.txt
docker images | grep mon-app >> validation-docker.txt
docker history mon-app:v1 | head -10 >> validation-docker.txt
echo "" >> validation-docker.txt
```

**Question de compréhension :** Pourquoi copie-t-on `requirements.txt` avant le reste du code ?

### **2.3 Test de l'image**

```bash
# Tester l'image créée
docker run -d --name test-app -p 5000:5000 mon-app:v1

# Attendre le démarrage
sleep 3

# Tester l'application
curl http://localhost:5000/health
curl http://localhost:5000
```

**VÉRIFICATION 6 :**
```bash
echo "=== ÉTAPE 2.3 : Test application ===" >> validation-docker.txt
docker logs test-app >> validation-docker.txt
curl -s http://localhost:5000/health >> validation-docker.txt
echo "" >> validation-docker.txt

# Nettoyer
docker stop test-app
docker rm test-app
```

### **2.4 Dockerfile optimisé avec sécurité**

Créez `Dockerfile.secure` :

```dockerfile
FROM python:3.9-slim

# Métadonnées
LABEL maintainer="votre-nom"
LABEL description="Application Flask sécurisée"
LABEL version="2.0"

# Variables d'environnement
ENV PYTHONUNBUFFERED=1
ENV FLASK_APP=app.py
ENV PORT=5000

# Créer un utilisateur non-root pour la sécurité
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Répertoire de travail
WORKDIR /app

# Copier les dépendances
COPY requirements.txt .

# Installer les dépendances
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code et ajuster les permissions
COPY . .
RUN chown -R appuser:appuser /app

# Basculer vers l'utilisateur non-root
USER appuser

# Port exposé
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# Commande par défaut
CMD ["python", "app.py"]
```

```bash
# Construire la version sécurisée
docker build -f Dockerfile.secure -t mon-app:secure .
```

**VÉRIFICATION 7 :**
```bash
echo "=== ÉTAPE 2.4 : Image sécurisée ===" >> validation-docker.txt
docker images | grep mon-app >> validation-docker.txt

# Tester la sécurité
docker run --rm mon-app:secure whoami >> validation-docker.txt
echo "" >> validation-docker.txt
```

**Question de compréhension :** Pourquoi est-il important d'utiliser un utilisateur non-root dans un conteneur ?

---

## **Étape 3 : Gestion des volumes et données (15 min)**

### **3.1 Comprendre l'éphémère**

```bash
# Lancer l'app et ajouter des données
docker run -d --name app-ephemere -p 5000:5000 mon-app:secure

# Ajouter quelques messages via l'interface web
# Ou via curl :
curl -X POST http://localhost:5000/add -d "message=Test Docker" -H "Content-Type: application/x-www-form-urlencoded"
curl -X POST http://localhost:5000/add -d "message=Deuxième message" -H "Content-Type: application/x-www-form-urlencoded"

# Vérifier les données
curl http://localhost:5000/ | grep "Test Docker"

# Redémarrer le conteneur
docker restart app-ephemere

# Vérifier si les données sont toujours là
curl http://localhost:5000/
```

**VÉRIFICATION 8 :**
```bash
echo "=== ÉTAPE 3.1 : Test persistance ===" >> validation-docker.txt
echo "Données après redémarrage :" >> validation-docker.txt
curl -s http://localhost:5000/ | grep -o "Test Docker\|Deuxième message" >> validation-docker.txt 2>/dev/null || echo "Données perdues" >> validation-docker.txt
echo "" >> validation-docker.txt

docker stop app-ephemere
docker rm app-ephemere
```

**Question de compréhension :** Pourquoi les données ont-elles disparu après le redémarrage ?

### **3.2 Volumes nommés**

```bash
# Créer un volume pour persister les données
docker volume create app-data

# Lister les volumes
docker volume ls

# Inspecter le volume
docker volume inspect app-data
```

### **3.3 Application avec persistance (Bind Mount)**

```bash
# Créer un dossier pour les logs
mkdir logs

# Lancer avec bind mount pour les logs
docker run -d --name app-persistent \
    -p 5000:5000 \
    -v $(pwd)/logs:/app/logs \
    -e DEBUG=true \
    mon-app:secure

# Vérifier le montage
docker inspect app-persistent | grep -A10 "Mounts"
```

**VÉRIFICATION 9 :**
```bash
echo "=== ÉTAPE 3.3 : Volumes et montages ===" >> validation-docker.txt
docker volume ls >> validation-docker.txt
docker inspect app-persistent | grep -A5 "Mounts" >> validation-docker.txt
ls -la logs/ >> validation-docker.txt 2>/dev/null || echo "Dossier logs non créé" >> validation-docker.txt
echo "" >> validation-docker.txt

docker stop app-persistent
docker rm app-persistent
```

---

## **Étape 4 : Réseaux et communication (10 min)**

### **4.1 Réseau personnalisé**

```bash
# Créer un réseau personnalisé
docker network create mon-reseau

# Lister les réseaux
docker network ls

# Inspecter le réseau
docker network inspect mon-reseau
```

### **4.2 Communication entre conteneurs**

```bash
# Lancer l'application sur le réseau personnalisé
docker run -d --name app-finale \
    --network mon-reseau \
    -p 5000:5000 \
    mon-app:secure

# Lancer un conteneur de test sur le même réseau
docker run -d --name testeur \
    --network mon-reseau \
    busybox sleep 300

# Tester la communication par nom
docker exec testeur ping -c 2 app-finale
docker exec testeur wget -qO- http://app-finale:5000/health
```

**🔍 VÉRIFICATION 10 :**
```bash
echo "=== ÉTAPE 4.2 : Communication réseau ===" >> validation-docker.txt
docker network ls | grep mon-reseau >> validation-docker.txt
docker exec testeur wget -qO- http://app-finale:5000/health >> validation-docker.txt
echo "" >> validation-docker.txt
```

**Question de compréhension :** Comment les conteneurs peuvent-ils se joindre par nom sur un réseau personnalisé ?

---

## **Étape 5 : Image finale et optimisations (5 min)**

### **5.1 Dockerfile final optimisé**

Créez `Dockerfile.final` :

```dockerfile
# Multi-stage build pour optimiser la taille
FROM python:3.9-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.9-slim as runtime

# Métadonnées complètes
LABEL maintainer="votre-nom"
LABEL description="Application Flask optimisée et sécurisée"
LABEL version="final"

# Variables d'environnement
ENV PYTHONUNBUFFERED=1
ENV FLASK_APP=app.py
ENV PORT=5000
ENV PATH=/home/appuser/.local/bin:$PATH

# Copier les dépendances du stage builder
COPY --from=builder /root/.local /home/appuser/.local

# Créer utilisateur non-root
RUN groupadd -r appuser && useradd -r -g appuser -d /home/appuser appuser

# Configuration de l'application
WORKDIR /app
COPY . .
RUN chown -R appuser:appuser /app

# Basculer vers utilisateur sécurisé
USER appuser

# Port et health check
EXPOSE 5000
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"

# Commande optimisée pour la production
CMD ["python", "app.py"]
```

```bash
# Construire l'image finale
docker build -f Dockerfile.final -t mon-app:final .

# Comparer les tailles
docker images | grep mon-app
```

**🔍 VÉRIFICATION FINALE :**
```bash
echo "=== ÉTAPE 5.1 : Image finale ===" >> validation-docker.txt
docker images | grep mon-app >> validation-docker.txt

# Test final complet
docker run -d --name app-final-test -p 5000:5000 mon-app:final
sleep 3

echo "Test final de l'application :" >> validation-docker.txt
curl -s http://localhost:5000/health >> validation-docker.txt
curl -s -X POST http://localhost:5000/add -d "message=Test final Docker" -H "Content-Type: application/x-www-form-urlencoded"
curl -s http://localhost:5000/ | grep -o "Test final Docker" >> validation-docker.txt
echo "" >> validation-docker.txt

# Informations sur l'image finale
echo "Informations image finale :" >> validation-docker.txt
docker inspect mon-app:final | grep -E "User|WorkingDir|ExposedPorts" >> validation-docker.txt
echo "" >> validation-docker.txt
```

---

## **Livrable et validation complète**

### **Nettoyage et finalisation**

```bash
# Arrêter tous les conteneurs
docker stop $(docker ps -q)

# Supprimer les conteneurs de test
docker rm $(docker ps -aq)

# Garder seulement l'image finale
docker rmi mon-app:v1 mon-app:secure 2>/dev/null || true

# Ajouter le récapitulatif final
echo "=== RÉCAPITULATIF FINAL ===" >> validation-docker.txt
echo "Image créée : mon-app:final" >> validation-docker.txt
echo "Taille : $(docker images mon-app:final --format 'table {{.Size}}' | tail -1)" >> validation-docker.txt
echo "Date de création : $(date)" >> validation-docker.txt
echo "Fonctionnalités validées :" >> validation-docker.txt
echo "✅ Manipulation des conteneurs" >> validation-docker.txt
echo "✅ Création d'images avec Dockerfile" >> validation-docker.txt
echo "✅ Optimisations de sécurité (utilisateur non-root)" >> validation-docker.txt
echo "✅ Gestion des volumes" >> validation-docker.txt
echo "✅ Réseaux Docker" >> validation-docker.txt
echo "✅ Multi-stage build" >> validation-docker.txt
echo "✅ Health checks" >> validation-docker.txt
```

---

## **Checklist de validation finale**

### **Livrables à fournir :**

1. **Image Docker** : `mon-app:final` créée et fonctionnelle
2. **Fichier de validation** : `validation-docker.txt` avec toutes les vérifications

### **Vérifications obligatoires :**

- [ ] L'image `mon-app:final` existe et fait moins de 200MB
- [ ] L'application répond sur http://localhost:5000
- [ ] Le health check fonctionne : http://localhost:5000/health
- [ ] L'image utilise un utilisateur non-root
- [ ] Le fichier `validation-docker.txt` contient toutes les vérifications
- [ ] Vous pouvez expliquer la différence entre image et conteneur
- [ ] Vous savez pourquoi l'ordre des instructions Dockerfile est important
- [ ] Vous comprenez l'utilité des volumes pour la persistance

### **Test de validation automatique**

```bash
# Script de test final
echo "🧪 TEST FINAL AUTOMATIQUE"
echo "========================="

# Test 1: Image existe
if docker images mon-app:final | grep -q "mon-app"; then
    echo "✅ Image mon-app:final créée"
else
    echo "❌ Image mon-app:final manquante"
fi

# Test 2: Application fonctionne
docker run -d --name validation-finale -p 5000:5000 mon-app:final
sleep 3

if curl -s http://localhost:5000/health | grep -q "ok"; then
    echo "✅ Application fonctionnelle"
else
    echo "❌ Application ne répond pas"
fi

# Test 3: Sécurité
USER_CHECK=$(docker exec validation-finale whoami 2>/dev/null)
if [ "$USER_CHECK" != "root" ]; then
    echo "✅ Utilisateur non-root configuré"
else
    echo "❌ Application tourne en root"
fi

# Test 4: Fichier de validation
if [ -f "validation-docker.txt" ]; then
    echo "✅ Fichier de validation présent"
    echo "   Lignes : $(wc -l < validation-docker.txt)"
else
    echo "❌ Fichier validation-docker.txt manquant"
fi

# Nettoyage final
docker stop validation-finale 2>/dev/null
docker rm validation-finale 2>/dev/null

echo ""
echo "🏆 ATELIER TERMINÉ !"
echo "Livrables : image 'mon-app:final' + fichier 'validation-docker.txt'"
```

---

## 💡 **Questions de réflexion finale**

1. **Expliquez en une phrase la différence entre une image et un conteneur.**

2. **Pourquoi Docker résout-il le problème "ça marche sur ma machine" ?**

3. **Quels sont les 3 avantages principaux des conteneurs par rapport aux VMs ?**

4. **Pourquoi est-il important d'utiliser un utilisateur non-root dans un conteneur ?**

5. **Dans quel ordre faut-il organiser les instructions d'un Dockerfile et pourquoi ?**

**Félicitations ! Vous maîtrisez maintenant les concepts fondamentaux de Docker !** 

