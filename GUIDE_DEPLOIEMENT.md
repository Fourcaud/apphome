# 🚀 Guide de Déploiement - Mes Super Missions

## 📋 Prérequis sur votre serveur

- **Node.js** version 18 ou supérieure
- **npm** (inclus avec Node.js)
- Accès SSH à votre serveur
- Un nom de domaine ou sous-domaine (ex: `missions.votresite.com`)

---

## 📁 Structure du projet

```
mes-super-missions/
├── backend/
│   ├── server.js          # Serveur API
│   ├── package.json       # Dépendances backend
│   ├── .env.example       # Configuration exemple
│   └── database.sqlite    # Base de données (créée auto)
└── frontend/
    ├── src/
    │   ├── App.js         # Application React
    │   ├── App.css        # Styles
    │   └── index.js       # Point d'entrée
    ├── public/
    │   └── index.html     # Page HTML
    └── package.json       # Dépendances frontend
```

---

## 🔧 Installation étape par étape

### 1. Transférer les fichiers sur votre serveur

```bash
# Depuis votre ordinateur, utilisez scp ou FileZilla
scp -r mes-super-missions/ user@votre-serveur:/var/www/
```

Ou connectez-vous en SSH et clonez/copiez les fichiers :
```bash
ssh user@votre-serveur
cd /var/www
mkdir mes-super-missions
# Puis transférez les fichiers
```

### 2. Installer les dépendances Backend

```bash
cd /var/www/mes-super-missions/backend
npm install
```

### 3. Configurer le Backend

```bash
# Copier le fichier de configuration
cp .env.example .env

# Éditer la configuration
nano .env
```

Contenu du fichier `.env` :
```
PORT=3001
NODE_ENV=production
```

### 4. Installer et construire le Frontend

```bash
cd /var/www/mes-super-missions/frontend
npm install
npm run build
```

Cette commande crée un dossier `build/` avec les fichiers optimisés.

### 5. Tester le serveur

```bash
cd /var/www/mes-super-missions/backend
node server.js
```

Vous devriez voir :
```
🚀 Serveur démarré sur le port 3001
📁 Base de données: /var/www/mes-super-missions/backend/database.sqlite
```

---

## 🔄 Configuration avec PM2 (Recommandé)

PM2 garde votre application en ligne 24/7 et la redémarre automatiquement.

### Installer PM2

```bash
npm install -g pm2
```

### Démarrer l'application

```bash
cd /var/www/mes-super-missions/backend
pm2 start server.js --name "mes-super-missions"
```

### Configurer le démarrage automatique

```bash
pm2 startup
pm2 save
```

### Commandes utiles PM2

```bash
pm2 status                    # Voir le statut
pm2 logs mes-super-missions   # Voir les logs
pm2 restart mes-super-missions # Redémarrer
pm2 stop mes-super-missions   # Arrêter
```

---

## 🌐 Configuration Nginx (Reverse Proxy)

Si vous avez déjà un site web sur votre serveur, vous utilisez probablement Nginx.

### Créer un fichier de configuration

```bash
sudo nano /etc/nginx/sites-available/missions
```

### Configuration pour un sous-domaine

```nginx
server {
    listen 80;
    server_name missions.votresite.com;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Configuration pour un sous-dossier (ex: votresite.com/missions)

```nginx
# Ajouter dans votre configuration existante
location /missions {
    rewrite ^/missions(.*)$ $1 break;
    proxy_pass http://localhost:3001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_cache_bypass $http_upgrade;
}
```

### Activer la configuration

```bash
sudo ln -s /etc/nginx/sites-available/missions /etc/nginx/sites-enabled/
sudo nginx -t                 # Tester la configuration
sudo systemctl reload nginx   # Appliquer
```

---

## 🔒 Ajouter HTTPS avec Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d missions.votresite.com
```

Certbot configurera automatiquement HTTPS.

---

## 🔥 Configuration Apache (Alternative à Nginx)

Si vous utilisez Apache au lieu de Nginx :

```bash
sudo a2enmod proxy proxy_http
sudo nano /etc/apache2/sites-available/missions.conf
```

```apache
<VirtualHost *:80>
    ServerName missions.votresite.com
    
    ProxyPreserveHost On
    ProxyPass / http://localhost:3001/
    ProxyPassReverse / http://localhost:3001/
</VirtualHost>
```

```bash
sudo a2ensite missions.conf
sudo systemctl reload apache2
```

---

## 📱 Accès à l'application

Une fois déployée :

- **Enfants** : `https://missions.votresite.com`
- **Parents** : Cliquer sur "Espace Parent" → Code PIN par défaut : `1234`

⚠️ **Important** : Changez le code PIN par défaut dans l'Espace Parent → Sécurité

---

## 🔧 Dépannage

### L'application ne démarre pas

```bash
# Vérifier les logs
pm2 logs mes-super-missions

# Vérifier que le port n'est pas utilisé
sudo lsof -i :3001
```

### Erreur de base de données

```bash
# Vérifier les permissions
ls -la /var/www/mes-super-missions/backend/
chmod 755 /var/www/mes-super-missions/backend/
```

### Nginx renvoie une erreur 502

```bash
# Vérifier que le backend tourne
pm2 status

# Redémarrer si nécessaire
pm2 restart mes-super-missions
```

### Réinitialiser la base de données

```bash
cd /var/www/mes-super-missions/backend
rm database.sqlite
pm2 restart mes-super-missions
# Les données par défaut seront recréées
```

---

## 📊 Sauvegardes

La base de données est un simple fichier SQLite. Pour sauvegarder :

```bash
# Sauvegarde manuelle
cp /var/www/mes-super-missions/backend/database.sqlite ~/backups/database_$(date +%Y%m%d).sqlite

# Sauvegarde automatique (cron)
crontab -e
# Ajouter cette ligne pour une sauvegarde quotidienne à 2h du matin :
0 2 * * * cp /var/www/mes-super-missions/backend/database.sqlite /home/user/backups/database_$(date +\%Y\%m\%d).sqlite
```

---

## 🆕 Mises à jour

Pour mettre à jour l'application :

```bash
cd /var/www/mes-super-missions

# Mettre à jour les fichiers (git pull ou scp)

# Backend
cd backend
npm install
pm2 restart mes-super-missions

# Frontend (si modifié)
cd ../frontend
npm install
npm run build
```

---

## 💡 Conseils

1. **Testez d'abord en local** avant de déployer
2. **Sauvegardez régulièrement** la base de données
3. **Changez le code PIN** par défaut immédiatement
4. **Utilisez HTTPS** pour la sécurité
5. **Surveillez les logs** avec `pm2 logs`

---

## 🆘 Support

En cas de problème :
1. Consultez les logs : `pm2 logs mes-super-missions`
2. Vérifiez la configuration Nginx : `sudo nginx -t`
3. Testez l'API directement : `curl http://localhost:3001/api/children`

Bonne utilisation ! 🎉
