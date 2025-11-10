# Guide de Déploiement LangGraph sur Railway (Gratuit)

## 🎯 Prérequis

1. Compte GitHub
2. Compte Railway (gratuit) : https://railway.app
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

### Étape 2 : Créer un compte Railway

1. Allez sur https://railway.app
2. Cliquez sur "Start a New Project"
3. Connectez votre compte GitHub

### Étape 3 : Créer les services nécessaires

#### 3.1 Créer Postgres

1. Dans Railway, cliquez sur "+ New"
2. Sélectionnez "Database" → "Add PostgreSQL"
3. Railway créera automatiquement une base de données
4. Notez les variables d'environnement (elles apparaîtront automatiquement)

#### 3.2 Créer Redis

1. Cliquez sur "+ New"
2. Sélectionnez "Database" → "Add Redis"
3. Notez l'URL de connexion Redis

### Étape 4 : Déployer votre application LangGraph

#### Option A : Déploiement depuis GitHub (Recommandé)

1. Dans Railway, cliquez sur "+ New"
2. Sélectionnez "GitHub Repo"
3. Choisissez votre repository contenant le projet LangGraph
4. Railway détectera automatiquement le Dockerfile

#### Option B : Déploiement depuis Dockerfile local

1. Dans Railway, cliquez sur "+ New"
2. Sélectionnez "Empty Project"
3. Cliquez sur "+ New" → "GitHub Repo" ou "Deploy from Dockerfile"

### Étape 5 : Configurer les variables d'environnement

Dans les paramètres de votre service LangGraph, ajoutez ces variables :

```bash
# Postgres (copiez depuis le service Postgres créé)
DATABASE_URI=postgresql://postgres:password@hostname:5432/railway

# Redis (copiez depuis le service Redis créé)
REDIS_URI=redis://default:password@hostname:6379

# LangSmith (gratuit)
LANGSMITH_API_KEY=lsv2_pt_...

# Optionnel : Port (Railway définit automatiquement PORT)
PORT=8000
```

**Comment obtenir les variables :**
- Cliquez sur le service Postgres → onglet "Variables" → copiez `DATABASE_URL`
- Cliquez sur le service Redis → onglet "Variables" → copiez `REDIS_URL`

### Étape 6 : Configurer le build

1. Dans les paramètres de votre service, allez dans "Settings"
2. Assurez-vous que "Root Directory" est correct (généralement `/` ou la racine)
3. Railway utilisera automatiquement le Dockerfile à la racine

### Étape 7 : Déployer

1. Railway déploiera automatiquement à chaque push sur GitHub
2. Ou cliquez sur "Deploy" pour un déploiement manuel
3. Attendez que le build se termine (2-5 minutes)

### Étape 8 : Obtenir l'URL de votre API

1. Une fois déployé, Railway génère automatiquement une URL
2. Cliquez sur votre service → onglet "Settings" → "Generate Domain"
3. Votre API sera accessible sur : `https://votre-projet.up.railway.app`

## ✅ Vérification

Testez votre API :

```bash
curl https://votre-projet.up.railway.app/ok
```

Vous devriez recevoir : `{"ok":true}`

## 🔧 Configuration avancée

### Utiliser un Dockerfile personnalisé

Si vous avez besoin de personnaliser le build :

1. Créez un `railway.json` à la racine :

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "startCommand": "langgraph start",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### Monitoring et logs

- **Logs** : Cliquez sur votre service → onglet "Deployments" → "View Logs"
- **Métriques** : Railway affiche automatiquement CPU, RAM, réseau

## 💰 Coûts

- **Gratuit** : $5 de crédit/mois (environ 500 heures)
- **Postgres** : Inclus dans le crédit gratuit
- **Redis** : Inclus dans le crédit gratuit
- **Bande passante** : 100 GB/mois inclus

## 🚨 Limitations du plan gratuit

- Services peuvent s'arrêter après inactivité (Railway les redémarre automatiquement)
- Limite de ressources (CPU/RAM) mais suffisant pour le développement
- Pas de haute disponibilité garantie

## 📚 Ressources

- [Documentation Railway](https://docs.railway.app)
- [Documentation LangGraph](https://langchain-ai.github.io/langgraph/)
- [Support Railway](https://railway.app/discord)

## 🆘 Dépannage

### Le build échoue
- Vérifiez que le Dockerfile est à la racine
- Vérifiez les logs de build dans Railway
- Assurez-vous que toutes les dépendances sont dans `requirements.txt` ou `pyproject.toml`

### L'application ne démarre pas
- Vérifiez que toutes les variables d'environnement sont définies
- Vérifiez les logs de déploiement
- Testez localement avec `langgraph dev` d'abord

### Erreur de connexion à Postgres/Redis
- Vérifiez que les services Postgres et Redis sont déployés
- Vérifiez que les variables d'environnement sont correctes
- Assurez-vous que les services sont dans le même projet Railway

