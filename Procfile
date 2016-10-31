nginx: nginx
confgen: docker-gen -notify-output -watch -only-exposed -notify "nginx -s reload" /app/nginx.tmpl /etc/nginx/conf.d/default.conf
sslgen: docker-gen -notify-output -watch -only-exposed -wait "500ms:2s" -notify "/createSSL.sh" /app/ssl.tmpl /createSSL.sh
discoverygen: docker-gen -watch /app/discovery.tmpl /var/www/html/discovery/index.html
cron: /usr/sbin/cron -f 
