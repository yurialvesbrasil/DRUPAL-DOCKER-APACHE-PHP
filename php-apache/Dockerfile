# from https://www.drupal.org/docs/8/system-requirements/drupal-8-php-requirements
FROM php:7.3-apache-stretch
# TODO switch to buster once https://github.com/docker-library/php/issues/865 is resolved in a clean way (either in the PHP image or in PHP itself)

# install the PHP extensions we need
RUN set -eux; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libzip-dev \
	; \
	\
	docker-php-ext-configure gd \
		--with-freetype-dir=/usr \
		--with-jpeg-dir=/usr \
		--with-png-dir=/usr \
	; \
	\
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

WORKDIR /var/www/html

# https://www.drupal.org/node/3060/release
#ENV DRUPAL_VERSION 8.9.x
#ENV DRUPAL_MD5 c0a0d2465788308f65773864d0af0a7e

#RUN set -eux; \
#	curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz; \
#	echo "${DRUPAL_MD5} *drupal.tar.gz" | md5sum -c -; \
#	tar -xz --strip-components=1 -f drupal.tar.gz; \
#	rm drupal.tar.gz; \
#	chown -R www-data:www-data sites modules themes
# vim:set ft=dockerfile:

# Para usar composer para instalar drupal
# Instalar: sudo apt-get install php7.2-curl
# Rodar na pasta raiz: ./composer.phar global require hirak/prestissimo
# Instalar: sudo apt install php-xml
# Instalar: sudo apt-get install php-mbstring
# Rodar: ./composer.phar update
# Rodar: ./composer.phar global require cviebrock/eloquent-sluggable
# Instalar: sudo apt-get install php7.2-gd
# Rodar na pasta raiz: ./composer.phar create-project drupal-composer/drupal-project:8.x-dev app --no-interaction
# Executa: mkdir -p app/config/sync/
# No fim da instalaçao rodar: chmod -R 555 app/web/sites/default/files/translations/
# No fim da instalaçao rodar: chmod 555 web/sites/default

#Configurações Pós-Instalação
# Um nível abaixo da pasta web, em meu-projeto/app/, renomeie .env.example para arquivo .env e adicione todas as credenciais:
# Uncomment and populate as needed.
 #MYSQL_DATABASE=dev
 #MYSQL_HOSTNAME=mariadb
 #MYSQL_PASSWORD=dev123
 #MYSQL_PORT=3306
 #MYSQL_USER=dev

 #E no arquivo sites/default/settings.php, remova/substitua por: 
 #$databases['default']['default'] = [
 #  'database' => getenv('MYSQL_DATABASE'),
 #  'driver' => 'mysql',
 #  'host' => getenv('MYSQL_HOSTNAME'),
 #  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
 #  'password' => getenv('MYSQL_PASSWORD'),
 #  'port' => getenv('MYSQL_PORT'),
 #  'prefix' => '',
 #  'username' => getenv('MYSQL_USER'),
 #];
 #Agora sim podemos alterar as permissões, rodando no terminal:
 #$ chmod 444 web/sites/default/settings.php 

#fonte: https://www.fabianomonteiro.com/como-instalar-o-drupal-composer-num-ambiente-de-desenvolvimento-docker

#Copia vhost.conf
RUN set -eux; \
    rm -f /etc/apache2/sites-enabled/000-default.conf
COPY vhosts.conf /etc/apache2/sites-enabled/vhosts.conf