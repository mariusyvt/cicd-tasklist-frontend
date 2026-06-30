# Task List Frontend - Docker Setup

## Configuration Docker

L'application est conteneurisée avec **Nginx** pour servir l'application React en production.

### Architecture

- **Build Stage** : Node.js 20 Alpine - compile l'app Vite
- **Runtime Stage** : Nginx Alpine - sert les fichiers statiques
- **Optimisations** : Compression Gzip, cache busting, SPA routing

## Build et Lancement

### Avec Docker Compose (Recommandé)

```bash
# Build et lancer
docker-compose up --build

# Accès
http://localhost
```

### Avec Docker CLI

```bash
# Build l'image
docker build -t tasklist-frontend:latest .

# Lancer le conteneur
docker run -d -p 80:80 --name tasklist-frontend tasklist-frontend:latest

# Arrêter
docker stop tasklist-frontend
docker rm tasklist-frontend
```

## Configuration Nginx

- **Port** : 80
- **Document Root** : /usr/share/nginx/html
- **SPA Routing** : Les routes non trouvées sont redirigées vers index.html
- **Gzip** : Activé pour texte, JS, CSS
- **Cache** :
  - Assets (js, css, images) : 1 an
  - HTML : No cache (validation à chaque fois)
- **Health Check** : `/health` endpoint disponible

## Logs et Debugging

```bash
# Voir les logs
docker-compose logs -f tasklist-frontend

# Accéder au conteneur
docker exec -it tasklist-frontend sh

# Tester la santé
curl http://localhost/health
```

## Taille de l'image

- Image de base nginx:alpine : ~40 MB
- Application compilée : ~100-150 KB
- Total : ~40 MB (très léger)

## Variables d'Environnement

Aucune variable d'env requise (l'app est une SPA statique). Si besoin à l'avenir :

- Créer un `env.template.js` servi par Nginx
- Injecter les variables via un script d'entrypoint

## Health Check

Le conteneur inclut un health check qui teste :

- Intervalle : 30s
- Timeout : 3s
- Retries : 3
- Start period : 40s

## Fichiers

- `Dockerfile` : Multi-stage build
- `nginx.conf` : Configuration Nginx
- `.dockerignore` : Exclusions du build context
- `docker-compose.yml` : Orchestration locale
