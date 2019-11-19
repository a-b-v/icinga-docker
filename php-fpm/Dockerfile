FROM ubuntu:bionic as builder
MAINTAINER Alexander Vlasov <vlasov@gmail.com>

ARG     mysql_host=db
ARG     mysql_database=icinga
ARG     mysql_icinga_user=icinga
ARG     mysql_icinga_pass=icingapass
ARG     mysql_webdatabase=icingaweb2
ARG     mysql_icingaweb_user=icingaweb
ARG     mysql_icingaweb_pass=icingawebpass
ARG     icingaweb_admin_user=iadmin
ARG	icinga_host=icinga
ARG	api_user=api
ARG	api_pass=apipass
ARG	mysql_dirdatabase=director
ARG	mysql_diruser=director
ARG	mysql_dirpassword=directorpass
ARG     dir_user=dir
ARG     dir_password=dirpass
ARG	graphite_webhost=nginx

ENV     MYSQL_HOST=$mysql_host
ENV     MYSQL_DATABASE=$mysql_database
ENV     MYSQL_ICINGA_USER=$mysql_icinga_user
ENV     MYSQL_ICINGA_PASS=$mysql_icinga_pass
ENV     MYSQL_WEBDATABASE=$mysql_webdatabase
ENV     MYSQL_WEBUSER=$mysql_icingaweb_user
ENV     MYSQL_WEBPASSWORD=$mysql_icingaweb_pass
ENV     ICINGAWEB_ADMIN_USER=$icingaweb_admin_user
ENV	ICINGA_HOST=$icinga_host
ENV	API_USER=$api_user
ENV	API_PASS=$api_pass
ENV	MYSQL_DIRDATABASE=$mysql_dirdatabase
ENV	MYSQL_DIRUSER=$mysql_diruser
ENV	MYSQL_DIRPASSWORD=$mysql_dirpassword
ENV     DIR_USER=$dir_user
ENV     DIR_PASSWORD=$dir_password
ENV	GRAPHITE_WEBHOST=$graphite_webhost

RUN	apt -qq update \
	&& apt install -y --no-install-recommends --no-install-suggests curl gnupg ca-certificates \
	&& curl -s https://packages.icinga.com/icinga.key | apt-key add - \
	&&. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
	echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
	/etc/apt/sources.list.d/${DIST}-icinga.list \
	&& echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
	/etc/apt/sources.list.d/${DIST}-icinga.list \
	&& apt -qq update \
	&& apt upgrade -y

RUN	DEBIAN_FRONTEND=noninteractive apt -y --no-install-recommends --no-install-suggests install php php-fpm php-curl php-posix php-sockets php-gettext php-intl php-imagick php-gd php-mbstring php-xml php-mysql locales language-pack-ru icingaweb2 icingaweb2-common icingaweb2-module-doc icingaweb2-module-monitoring icingacli git openssl

RUN	icingacli module enable setup

RUN	icingacli setup config directory

RUN     echo "[icingaweb2]\n\
        type = \"db\"\n\
	db = \"mysql\"\n\
	host = \"$MYSQL_HOST\"\n\
	port = \"3306\"\n\
	dbname = \"$MYSQL_WEBDATABASE\"\n\
	username = \"$MYSQL_WEBUSER\"\n\
	password = \"$MYSQL_WEBPASSWORD\"\n\
	[director]\n\
        type = \"db\"\n\
        db = \"mysql\"\n\
        host = \"$MYSQL_HOST\"\n\
        port = \"3306\"\n\
        dbname = \"$MYSQL_DIRDATABASE\"\n\
        username = \"$MYSQL_DIRUSER\"\n\
        password = \"$MYSQL_DIRPASSWORD\"\n\
        charset = "utf8" \n\
	[icinga2]\n\
        type = \"db\"\n\
        db = \"mysql\"\n\
        host = \"$MYSQL_HOST\"\n\
        port = \"3306\"\n\
        dbname = \"$MYSQL_DATABASE\"\n\
        username = \"$MYSQL_ICINGA_USER\"\n\
        password = \"$MYSQL_ICINGA_PASS\"\n"\
        > /etc/icingaweb2/resources.ini

RUN	echo "[logging]\n\
        log = \"file\"\n\
        level = \"DEBUG\"\n\
	file = \"/var/log/icingaweb2.log\"\n\
        application = \"icingaweb2\"\n\
	[preferences]\n\
	resource = \"icingaweb2\"\n"\
	> /etc/icingaweb2/config.ini

RUN	echo "[icingaweb2]\n\
	backend = \"db\"\n\
	resource = \"icingaweb2\"\n"\
	> /etc/icingaweb2/authentication.ini

RUN	echo "[admins]\n\
	users = \"$ICINGAWEB_ADMIN_USER\"\n\
	permissions = \"*\"\n"\
	> /etc/icingaweb2/roles.ini

RUN     mkdir -p /etc/icingaweb2/modules/director/

RUN     echo "[db]\n\
        resource = "director"\n"\
        > /etc/icingaweb2/modules/director/config.ini

RUN	echo "[config]\n\
	endpoint = $ICINGA_HOST\n\
	host = $ICINGA_HOST\n\
	port = 5665\n\
        username = \"$DIR_USER\"\n\
        password = \"$DIR_PASSWORD\"\n"\
	> /etc/icingaweb2/modules/director/kickstart.ini

RUN	MODULE_NAME=ipl ; \
	MODULE_VERSION=v0.4.0 ;\
	MODULES_PATH="/usr/share/icingaweb2/modules" ;\
	REPO="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}" ;\
	git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}" \
	&& icingacli module enable "${MODULE_NAME}"

RUN     MODULE_NAME=incubator ; \
        MODULE_VERSION=v0.5.0 ;\
        MODULES_PATH="/usr/share/icingaweb2/modules" ;\
        REPO="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}" ;\
        git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}" \
        && icingacli module enable "${MODULE_NAME}"

RUN     MODULE_NAME=reactbundle ; \
        MODULE_VERSION=v0.7.0 ;\
        MODULES_PATH="/usr/share/icingaweb2/modules" ;\
        REPO="https://github.com/Icinga/icingaweb2-module-${MODULE_NAME}" ;\
        git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}" \
        && icingacli module enable "${MODULE_NAME}"

RUN	MODULE_NAME=director ; \
	MODULE_VERSION="v1.7.1" ; \
	MODULES_PATH="/usr/share/icingaweb2/modules" ; \
	REPO="https://github.com/icinga/icingaweb2-module-${MODULE_NAME}" ; \
        git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}" \
        && icingacli module enable "${MODULE_NAME}"

RUN	MODULE_NAME=graphite ; \
	MODULE_VERSION="v1.1.0" ; \
	MODULES_PATH="/usr/share/icingaweb2/modules" ; \
	REPO="https://github.com/icinga/icingaweb2-module-${MODULE_NAME}" ; \
        git clone ${REPO} "${MODULES_PATH}/${MODULE_NAME}" --branch "${MODULE_VERSION}" \
        && icingacli module enable "${MODULE_NAME}"

RUN	mkdir -p /etc/icingaweb2/modules/monitoring/

RUN     echo "[security]\n\
	protected_customvars = "*pw*,*pass*,community"\n"\
	> /etc/icingaweb2/modules/monitoring/config.ini

RUN     echo "[icinga2]\n\
	type = "ido"\n\
	resource = "icinga2"\n"\
	> /etc/icingaweb2/modules/monitoring/backends.ini

RUN     echo "[icinga2]\n\
	transport = "api"\n\
	host = "$ICINGA_HOST"\n\
	port = "5665"\n\
	username = "$API_USER"\n\
	password = "$API_PASS"\n"\
	> /etc/icingaweb2/modules/monitoring/commandtransports.ini

RUN     mkdir -p /etc/icingaweb2/modules/graphite

RUN	echo "[graphite]\n\
	url = \"http://$GRAPHITE_WEBHOST:81\"\n\
	insecure = \"1\"\n"\
	> /etc/icingaweb2/modules/graphite/config.ini
	
RUN	icingacli module enable monitoring

RUN	icingacli module enable translation

RUN	icingacli module enable graphite

RUN	mkdir -p /etc/nginx/sites-enabled/\
	&& mkdir -p /etc/nginx/certs/

RUN	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/certs/localhost.key -out /etc/nginx/certs/localhost.crt -subj /C=WO/ST=WO/L=CITY/O=Company/OU=Monitoring/CN=localhost\
	&& chown -R www-data:www-data /etc/nginx/certs/

RUN     echo "server {\n\
        listen 80; \n\
	listen 443 ssl;\n\
        server_name localhost;\n\
	ssl_protocols TLSv1.2 TLSv1.1 TLSv1;\n\
	ssl_certificate /etc/nginx/certs/localhost.crt;\n\
        ssl_certificate_key /etc/nginx/certs/localhost.key;\n\
	if (\$ssl_protocol = \"\") {rewrite ^ https://\$server_name\$request_uri? permanent;}\n\
        root /usr/share/icingaweb2;\n\
        location / {\n\
            return 301 /icingaweb2/;\n\
        }\n"\
        > /etc/nginx/sites-enabled/default

RUN	icingacli setup config webserver nginx --document-root /usr/share/icingaweb2/public --config /etc/icingaweb2 >> /etc/nginx/sites-enabled/default

RUN     echo "}\n" >> /etc/nginx/sites-enabled/default

RUN	sed -i \
	-e 's/fastcgi_pass 127.0.0.1:9000;/fastcgi_pass php-fpm:9000;/' \
	/etc/nginx/sites-enabled/default

RUN	echo "upstream graphite {\n\
	server graphite:8080 fail_timeout=0;\n\
	}\n\
	server {\n\
        listen 81;\n\
	server_name localhost;\n\
	root /usr/share/graphite/webapp;\n\
    	access_log /var/log/nginx/graphite.access.log;\n\
    	error_log  /var/log/nginx/graphite.error.log;\n\
    	location = /favicon.ico {\n\
           return 204;\n\
    	}\n\
    	location /static {\n\  
           alias /usr/share/graphite/webapp/content;\n\
           expires max;\n\
    	}\n\
    	location / {\n\
           try_files \$uri @graphite;\n\
    	}\n\
    	location @graphite {\n\
           proxy_pass_header Server;\n\
           proxy_set_header Host \$http_host;\n\
           proxy_redirect off;\n\
           proxy_set_header X-Real-IP \$remote_addr;\n\
           proxy_set_header X-Scheme \$scheme;\n\
           proxy_connect_timeout 10;\n\
           proxy_read_timeout 10;\n\
           proxy_pass http://graphite;\n\
    	}\n\
	}\n"\
	>> /etc/nginx/sites-enabled/graphite

RUN	DEBIAN_FRONTEND=noninteractive apt -y purge git openssl && apt -y autoremove && apt clean

RUN	sed -i "s/listen = \/run\/php\/php7.2-fpm.sock/listen = 9000/" /etc/php/7.2/fpm/pool.d/www.conf

RUN	mkdir /run/php/ \ 
	&& chown www-data:www-data /run/php/

RUN	touch /var/log/icingaweb2.log \
	&& chown www-data:www-data /var/log/icingaweb2.log


RUN	ln -sf /dev/stderr /var/log/php7.2-fpm.log

RUN	echo "#!/usr/bin/env bash\n\
	CONTAINER_ALREADY_STARTED=\"/var/spool/setupcompleted\"\n\
	if [ ! -f \$CONTAINER_ALREADY_STARTED ]; then\n\
	touch \$CONTAINER_ALREADY_STARTED\n\
	sleep 5\n\
	echo '-- Director initial setup started --'\n\
	icingacli director migration run --verbose\n\
	icingacli director kickstart run\n\
	echo '-- Director initial setup completed --'\n\
	fi\n\
	/usr/sbin/php-fpm7.2 -F --fpm-config=/etc/php/7.2/fpm/php-fpm.conf\n"\
	> /var/opt/start.sh

CMD	sh /var/opt/start.sh
