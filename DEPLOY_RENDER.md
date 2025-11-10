# Guide de Déploiement LangGraph sur Render (Gratuit)

## 🎯 Prérequis

1. Compte GitHub
2. Compte Render (gratuit) : https://render.com
3. Compte LangSmith (gratuit) : https://smith.langchain.com
4. LangGraph CLI installé localement

## 📋 Étapes de Déploiement

### Étape 1 : Préparer votre projet LangGraph

```bash
# 1. Créer un nouveau projet LangGraph (si vous n'en avez pas)
langgraph new mon-projet-langgraph --template new-langgraph-project-python

# 2. Aller dans le dossier du projet
cd mon-projet-langgraph

# 3. Générer le Dockerfile
langgraph dockerfile Dockerfile --add-docker-compose

# 4. Tester localement d'abord
langgraph dev
```

### Étape 2 : Créer un compte Render

1. Allez sur https://render.com
2. Cliquez sur "Get Started for Free"
3. Connectez votre compte GitHub
4. Vérifiez votre email

### Étape 3 : Créer les services nécessaires

#### 3.1 Créer Postgres

1. Dans le dashboard Render, cliquez sur "New +"
2. Sélectionnez "PostgreSQL"
3. Configurez :
   - **Name** : `langgraph-postgres` (ou votre nom)
   - **Database** : `langgraph` (ou laissez par défaut)
   - **User** : `langgraph` (ou laissez par défaut)
   - **Region** : Choisissez la région la plus proche
   - **PostgreSQL Version** : 16 (recommandé)
   - **Plan** : **Free** (gratuit pendant 90 jours, puis $7/mois)
4. Cliquez sur "Create Database"
5. ⚠️ **Important** : Notez que Postgres gratuit expire après 90 jours

#### 3.2 Créer Redis

1. Cliquez sur "New +"
2. Sélectionnez "Redis"
3. Configurez :
   - **Name** : `langgraph-redis` (ou votre nom)
   - **Region** : Même région que Postgres
   - **Plan** : **Free** (25 MB, suffisant pour le développement)
4. Cliquez sur "Create Redis"
5. ✅ Redis gratuit est permanent (pas d'expiration)

### Étape 4 : Déployer votre application LangGraph

#### Option A : Déploiement avec Dockerfile (Recommandé)

1. Dans Render, cliquez sur "New +"
2. Sélectionnez "Web Service"
3. Connectez votre repository GitHub
4. Configurez :
   - **Name** : `langgraph-api` (ou votre nom)
   - **Region** : Même région que vos bases de données
   - **Branch** : `main` (ou votre branche)
   - **Root Directory** : `/` (ou le dossier racine de votre projet)
   - **Runtime** : **Docker**
   - **Dockerfile Path** : `Dockerfile` (ou le chemin vers votre Dockerfile)
   - **Docker Context** : `/` (généralement la racine)
   - **Plan** : **Free** (512 MB RAM, 0.5 CPU)
5. Cliquez sur "Create Web Service"

#### Option B : Déploiement sans Docker (Alternative)

Si vous préférez ne pas utiliser Docker :

1. Créez un "Web Service" comme ci-dessus
2. Configurez :
   - **Runtime** : **Python 3**
   - **Build Command** : `pip install -e . "langgraph-cli[inmem]"`
   - **Start Command** : `langgraph dev --port $PORT --host 0.0.0.0`
   - **Python Version** : 3.11 ou supérieur

### Étape 5 : Configurer les variables d'environnement

Dans les paramètres de votre service web, allez dans "Environment" et ajoutez :

#### Variables requises

```bash
# Postgres - Obtenez depuis le service Postgres
DATABASE_URI=postgresql://user:password@hostname:5432/database

# Redis - Obtenez depuis le service Redis  
REDIS_URI=redis://red-xxxxx:6379

# LangSmith (gratuit)
LANGSMITH_API_KEY=lsv2_pt_...

# Port (Render définit automatiquement PORT)
PORT=10000
```

**Comment obtenir les variables :**

1. **DATABASE_URI** :
   - Allez dans votre service Postgres
   - Cliquez sur "Connections"
   - Copiez "Internal Database URL" ou construisez l'URI avec les infos affichées
   - Format : `postgresql://user:password@hostname:5432/database`

2. **REDIS_URI** :
   - Allez dans votre service Redis
   - Cliquez sur "Connections"
   - Copiez "Internal Redis URL"
   - Format : `redis://red-xxxxx:6379` ou `redis://:password@hostname:6379`

3. **LANGSMITH_API_KEY** :
   - Créez un compte sur https://smith.langchain.com
   - Allez dans Settings → API Keys
   - Créez une nouvelle clé API

#### Variables optionnelles

```bash
# Pour le développement (optionnel)
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=langgraph-render

# Si vous utilisez un domaine personnalisé
DOMAIN=votre-domaine.com
```

### Étape 6 : Configurer le build (si Docker)

Si vous utilisez Docker, Render détectera automatiquement le Dockerfile. Vérifiez que :

1. Le Dockerfile est à la racine du projet
2. Le "Dockerfile Path" dans Render pointe vers `Dockerfile`
3. Le "Docker Context" est correct (généralement `/`)

### Étape 7 : Déployer

1. Render déploiera automatiquement à chaque push sur GitHub
2. Ou cliquez sur "Manual Deploy" → "Deploy latest commit"
3. Attendez que le build se termine (3-7 minutes pour la première fois)
4. Surveillez les logs en temps réel

### Étape 8 : Obtenir l'URL de votre API

1. Une fois déployé, Render génère automatiquement une URL
2. Format : `https://votre-service.onrender.com`
3. L'URL est visible dans le dashboard de votre service
4. ⚠️ **Note** : Sur le plan gratuit, le service peut s'endormir après 15 minutes d'inactivité (première requête sera lente)

## ✅ Vérification

Testez votre API :

```bash
curl https://votre-service.onrender.com/ok
```

Vous devriez recevoir : `{"ok":true}`

## 🔧 Configuration avancée

### Utiliser render.yaml (Configuration as Code)

Créez un fichier `render.yaml` à la racine de votre projet :

```yaml
services:
  - type: web
    name: langgraph-api
    runtime: docker
    dockerfilePath: ./Dockerfile
    dockerContext: .
    plan: free
    envVars:
      - key: DATABASE_URI
        fromDatabase:
          name: langgraph-postgres
          property: connectionString
      - key: REDIS_URI
        fromService:
          name: langgraph-redis
          type: redis
          property: connectionString
      - key: LANGSMITH_API_KEY
        sync: false  # À définir manuellement dans le dashboard
      - key: PORT
        value: 10000

databases:
  - name: langgraph-postgres
    databaseName: langgraph
    user: langgraph
    plan: free
    postgresMajorVersion: 16

redis:
  - name: langgraph-redis
    plan: free
```

Puis dans Render :
1. Créez un "Blueprint"
2. Connectez votre repo GitHub
3. Render utilisera automatiquement `render.yaml`

### Configuration Docker personnalisée

Si vous avez besoin de personnaliser le build, modifiez votre Dockerfile :

```dockerfile
FROM langchain/langgraph-api:latest

# Votre code personnalisé
ADD . /deps/agent
WORKDIR /deps/agent

# Installer les dépendances
RUN pip install -e .

# Variables d'environnement
ENV LANGSERVE_GRAPHS='{"agent":"./src/graph.py:graph"}'

# Port
EXPOSE 8000

# Commande de démarrage
CMD ["langgraph", "start", "--port", "8000", "--host", "0.0.0.0"]
```

### Monitoring et logs

- **Logs** : Cliquez sur votre service → onglet "Logs" (logs en temps réel)
- **Métriques** : Render affiche automatiquement CPU, RAM, réseau
- **Events** : Historique des déploiements et événements

### Health Checks

Render vérifie automatiquement la santé de votre service. Assurez-vous que :

1. Votre application répond sur le port défini par `$PORT`
2. L'endpoint `/ok` ou `/health` retourne un statut 200

## 💰 Coûts

### Plan Gratuit

- **Web Service** : Gratuit (512 MB RAM, 0.5 CPU)
  - ⚠️ S'endort après 15 min d'inactivité
  - Redémarre automatiquement à la première requête (peut prendre 30-60 secondes)
  
- **Postgres** : Gratuit pendant **90 jours**, puis $7/mois
  - 1 GB de stockage
  - Backups quotidiens (sur plan payant)
  
- **Redis** : Gratuit (permanent)
  - 25 MB de stockage
  - Suffisant pour le développement

### Plan Starter ($7/mois)

Si vous voulez éviter l'endormissement :
- Web Service : $7/mois (512 MB RAM, toujours actif)
- Postgres : $7/mois (après 90 jours)
- Redis : Gratuit

## 🚨 Limitations du plan gratuit

### Web Service
- ⚠️ **Endormissement** : Le service s'endort après 15 minutes d'inactivité
- ⚠️ **Temps de démarrage** : 30-60 secondes au réveil
- Limite de ressources (512 MB RAM, 0.5 CPU)
- Pas de haute disponibilité garantie
- Bande passante limitée

### Postgres
- ⚠️ **Expiration** : Gratuit seulement 90 jours, puis $7/mois
- Pas de backups automatiques sur le plan gratuit
- 1 GB de stockage maximum

### Redis
- 25 MB de stockage (suffisant pour le développement)
- Pas de persistance sur le plan gratuit

## 🔄 Solutions pour éviter l'endormissement

### Option 1 : Service de ping (Gratuit)

Utilisez un service gratuit comme :
- **UptimeRobot** : https://uptimerobot.com (gratuit, 50 monitors)
- **Cron-job.org** : https://cron-job.org (gratuit)

Configurez un ping toutes les 5 minutes vers votre URL :
```
https://votre-service.onrender.com/ok
```

### Option 2 : Upgrade vers Starter ($7/mois)

Le plan Starter garde votre service toujours actif.

## 📚 Ressources

- [Documentation Render](https://render.com/docs)
- [Documentation LangGraph](https://langchain-ai.github.io/langgraph/)
- [Support Render](https://community.render.com)
- [Render Status](https://status.render.com)

## 🆘 Dépannage

### Le build échoue

**Problème** : Erreur lors du build Docker
- Vérifiez que le Dockerfile est à la racine
- Vérifiez les logs de build dans Render
- Assurez-vous que toutes les dépendances sont dans `requirements.txt` ou `pyproject.toml`
- Vérifiez que le "Docker Context" est correct

**Solution** :
```bash
# Testez localement d'abord
docker build -t test-langgraph .
docker run -p 8000:8000 test-langgraph
```

### L'application ne démarre pas

**Problème** : Service en "Unhealthy" ou erreur au démarrage
- Vérifiez que toutes les variables d'environnement sont définies
- Vérifiez les logs de déploiement dans Render
- Testez localement avec `langgraph dev` d'abord
- Vérifiez que le port est correct (`$PORT` ou `10000`)

**Solution** :
```bash
# Testez localement avec les mêmes variables
export DATABASE_URI=...
export REDIS_URI=...
export LANGSMITH_API_KEY=...
langgraph dev
```

### Erreur de connexion à Postgres/Redis

**Problème** : "Connection refused" ou "Connection timeout"
- Vérifiez que les services Postgres et Redis sont déployés
- Vérifiez que vous utilisez les URLs **internes** (pas externes)
- Assurez-vous que les services sont dans la même région
- Vérifiez le format des URIs

**Solution** :
- Utilisez "Internal Database URL" pour Postgres
- Utilisez "Internal Redis URL" pour Redis
- Format Postgres : `postgresql://user:password@hostname:5432/database`
- Format Redis : `redis://hostname:6379` ou `redis://:password@hostname:6379`

### Le service s'endort trop souvent

**Problème** : Service inactif après 15 minutes
- C'est normal sur le plan gratuit
- Utilisez un service de ping (UptimeRobot, etc.)
- Ou upgrade vers le plan Starter ($7/mois)

### Postgres expire après 90 jours

**Problème** : Postgres gratuit expire
- C'est normal, le plan gratuit Postgres dure 90 jours
- Options :
  1. Upgrade vers $7/mois pour Postgres
  2. Migrer vers une autre base de données gratuite (Supabase, Neon, etc.)
  3. Utiliser SQLite (non recommandé pour la production)

## 🎯 Checklist de déploiement

- [ ] Projet LangGraph testé localement
- [ ] Dockerfile généré avec `langgraph dockerfile`
- [ ] Compte Render créé
- [ ] Service Postgres créé (notez l'URL interne)
- [ ] Service Redis créé (notez l'URL interne)
- [ ] Service Web créé (Docker)
- [ ] Variables d'environnement configurées :
  - [ ] `DATABASE_URI`
  - [ ] `REDIS_URI`
  - [ ] `LANGSMITH_API_KEY`
  - [ ] `PORT` (optionnel)
- [ ] Déploiement réussi
- [ ] Test de l'endpoint `/ok` fonctionne
- [ ] Service de ping configuré (optionnel, pour éviter l'endormissement)

## 💡 Conseils

1. **Testez localement d'abord** : Assurez-vous que `langgraph dev` fonctionne avant de déployer
2. **Utilisez les URLs internes** : Pour Postgres et Redis, utilisez toujours les URLs internes
3. **Surveillez les logs** : Les logs Render sont très utiles pour le débogage
4. **Configurez un ping** : Utilisez UptimeRobot pour éviter l'endormissement
5. **Planifiez la migration Postgres** : Après 90 jours, vous devrez payer $7/mois ou migrer

