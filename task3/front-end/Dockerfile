# Utiliser la dernière version de Nginx
FROM nginx:latest

# Copier tous les fichiers du front-end vers le dossier web de Nginx
COPY ./softy-pinko-front-end /var/www/html/softy-pinko-front-end

# Copier la configuration Nginx personnalisée
COPY ./softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf

# Exposer le port 9000 (informatif)
EXPOSE 9000

# Nginx démarre automatiquement avec l'image, pas besoin de CMD