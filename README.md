# Practica-bitnami/Prestashop  Jose Francisco León López
## En está práctica vamos a crear un docker-compose que nos instale prestashop en nuestro dominio
### Para ello antes de empezar debemos instalar docker-compose y docker
#### El comando para instalar docker es:
~~~
sudo apt install docker.io
~~~
#### NOTA: Si no hacemos un apt update primero a la máquina no nos va a reconocer los comandos
#### El comando para instalar docker-compose es:
~~~
sudo apt install docker-compose
~~~
#### Una vez instalados los comandos creamos nuestro archivo de docker-compose:
~~~
version: '3.3'

services:
  prestashop:
    image: bitnami/prestashop:1.7
    environment:
      - PRESTASHOP_FIRST_NAME=${PRESTASHOP_FIRST_NAME}
      - PRESTASHOP_LAST_NAME=${PRESTASHOP_LAST_NAME}
      - PRESTASHOP_PASSWORD=${PRESTASHOP_PASSWORD}
      - PRESTASHOP_HOST=${DNS_DOMAIN_SECURE}
      - PRESTASHOP_EMAIL=${PRESTASHOP_EMAIL}
      - PRESTASHOP_COUNTRY=${PRESTASHOP_COUNTRY}
      - PRESTASHOP_DATABASE_HOST=mysql
      - PRESTASHOP_DATABASE_NAME=${PRESTASHOP_DATABASE_NAME}
      - PRESTASHOP_DATABASE_USER=${PRESTASHOP_DATABASE_USER}
      - PRESTASHOP_DATABASE_PASSWORD=${PRESTASHOP_DATABASE_PASSWORD}
      - PRESTASHOP_DATABASE_PREFIX=${PRESTASHOP_DATABASE_PREFIX}
      - PRESTASHOP_ENABLE_SSL=1
      - PRESTASHOP_ENABLE_HTTPS=yes
    volumes:
      - prestashop_data:/bitnami/prestashop
    depends_on:
      - mysql
    restart: always
    networks:
      - frontend_network
      - backend_network
  
  mysql:
    image: mysql:8.0
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${PRESTASHOP_DATABASE_NAME}
      - MYSQL_USER=${PRESTASHOP_DATABASE_USER}
      - MYSQL_PASSWORD=${PRESTASHOP_DATABASE_PASSWORD}
    restart: always
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend_network
    security_opt:
      - seccomp:unconfined

  phpmyadmin:
    image: phpmyadmin:5
    ports:
      - 8080:80
    environment:
      - PMA_HOST=mysql
    depends_on:
      - mysql
    restart: always
    networks:
      - frontend_network
      - backend_network

  https-portal:
    image: steveltn/https-portal:1
    ports:
      - 80:80
      - 443:443
    restart: always
    environment:
      DOMAINS: '${DNS_DOMAIN_SECURE} -> http://prestashop:8080'
      STAGE: 'production' 
    volumes:
      - ssl_certs_data:/var/lib/https-portal
    depends_on:
      - prestashop
    networks:
      - frontend_network

volumes:
  prestashop_data:
  mysql_data:
  ssl_certs_data:

networks:
  frontend_network:
  backend_network:
~~~
### Y el archivo de variables será el siguiente
~~~
# Aquí alojamos las variables de entorno de prestashop, el dominio y la contraseña de mysql
DNS_DOMAIN_SECURE=bitnami-prestashop.ddns.net
PRESTASHOP_FIRST_NAME=Bitnami
PRESTASHOP_LAST_NAME=User
PRESTASHOP_PASSWORD=bitnami1
PRESTASHOP_EMAIL=prestashop@prestashop.com
PRESTASHOP_COUNTRY=es
PRESTASHOP_LANGUAGE=es
PRESTASHOP_DATABASE_HOST=mysql
PRESTASHOP_DATABASE_NAME=prestashop
PRESTASHOP_DATABASE_USER=ps_user
PRESTASHOP_DATABASE_PASSWORD=ps_passwd
PRESTASHOP_DATABASE_PREFIX=ps_
MYSQL_ROOT_PASSWORD=root
~~~
### Cosas relevantes a la hora de hacer el docker-compose
### En el docker hemos instalado bitnami/prestashop, phpmyadmin, mysql y https-portal para que tenga el certificado de lets encrypt 
#### IMPORTANTE: 
#### - No cambiar los valores de las variables de dockerhub ya que si no no nos funcionará 
#### - Poner el stage del https portal en production por que si no no va a verificar el certificado
### Una vez hecho todo esto ejecutamos el comando:
~~~
sudo docker-compose up -d
~~~
### Para levantar el contenedor y ya nos saldrá en nuestro dominio nuestro prestashop con bitnami.
