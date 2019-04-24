# Nextcloud en Hosting.cl

## Instalación
- Instalar Vim, Git y Docker con `yum install -y vim git docker`
- Instalar Docker Compose con `curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose`
- Dejar SELinux en modo permisivo.. Primero temporalmente con `setenforce 0` y luego editando el archivo `vim /etc/selinux/config` y dejando `SELINUX=permissive` para mantener los cambios al reinicar el servidor.
- Habilitar el servicio de Docker para que se ejecute la reiniciar el servidor `systemctl enable docker` 
- Descargar el repositorio con `git clone https://github.com/tikoflano/nextcloud.git /home/nextcloud`
- Se debe habilitar la comunicación entre contenedores en el firewall con `firewall-cmd --permanent --zone=public --add-rich-rule='rule family=ipv4 source address=172.20.0.0/16 accept'`, luego reiniciar el firewall y luego reiniciar docker con `systemctl restart firewalld && systemctl restart docker`.
- Copiar el archivo de configuración de ejemplo `cp /home/nextcloud/example.env /home/nextcloud/.env`
- Editar el archivo de configuracion `vim /home/nextcloud/.env` con los valores que se quieran usar

- Ejecutar `docker-compose -f docker-compose.yml [-f docker-compose.collabora.yml] [-f docker-compose.onlyoffice.yml] up -d` según si se quiere agregar Collabora y/o OnlyOffice. Luego se puede ingresar a https://<NEXTCLOUD_SUBDOMAIN>.<DOMAIN>

- Ejecutar `docker exec --user www-data nextcloud php occ db:convert-filecache-bigint` para evitar un aviso que sale en el estado del sistema.
- Cambiar le modo de ejecución de los trabajos en segundo plano. Se debe dejar el modo *cron* con el comando `docker-compose exec -u www-data nextcloud php occ background:cron`

## Habilitar Collabora
Luego de instalar la app, se debe usar la URL https://<COLLABORA_SUBDOMAIN>.<DOMAIN> en la configuración. Si aparece un mensaje diciendo *Saved with error* se puede ignorar.

Para comprobar si está ejecutándose se puede ingresar a https://<COLLABORA_SUBDOMAIN>.<DOMAIN>/loleaflet/dist/admin/admin.html
  
## Habilitar OnlyOffice
Luego de instalar la app, se debe usar la siguiente configuración (habilitar configuración avanzada):
  - *Document Editing Service address*: https://<ONLYOFFICE_SUBDOMAIN>.<DOMAIN>
  - *Document Editing Service address for internal requests from the server*: https://<ONLYOFFICE_SUBDOMAIN>.<DOMAIN>
  - *Server address for internal requests from the Document Editing Service*: https://<NEXTCLOUD_SUBDOMAIN>.<DOMAIN>
  
Para comprobar si está ejecutándose se puede ingresar a https://<ONLYOFFICE_SUBDOMAIN>.<DOMAIN>
  
## Usar certificado propio
- Eliminar la variables de entorno `LETSENCRYPT_*` del archivo .
- Copiar en `/var/lib/docker/volumes/nextcloud_proxy-certs/_data/` los archivos .crt y .key que componen el certificado. El nombre de estos archivos debe ser exactamente igual al nombre del `VIRTUAL_HOST` del servicio, terminado con .crt y .key

Con esto el contendor proxy generará el virtualhost correspondiente para que use el certificado.
  
## Errores comunes
1. Al entrar al sitio aparece como no seguro. Luego al ver el certificado en el navegador dice emitido por y para *letsencrypt-nginx-proxy-companion*.
  Esto ocurre porque el servicio que provee los ceritificados aun no lo ha emitido. Posibles razones:
  - Aun esta trabajando en eso. Puede tardar unos 5 minutos.
  - El subdominio nextcloud.dominio.tld aun no responde públicamente a la IP del servidor.
  - Se ha alcanzado el límite de certificados gratuitos posibles para emitir por Let's Encrypt (https://letsencrypt.org/docs/rate-limits/). 
2. **502 Bad Gateway**
Alguno de los servicios aún no arranca, hay que esperar unos 5 minutos. En caso de persistir el problema se deben ver los logs.
