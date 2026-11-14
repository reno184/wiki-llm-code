# Copier depuis le conteneur vers l'hôte
docker cp mon-conteneur:/usr/share/nginx/html ./dist-copie

# Ou monter un volume si vous voulez un accès permanent
docker run -v /srv/mon-site:/usr/share/nginx/html ...
