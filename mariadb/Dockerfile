FROM mariadb:10.4 as builder

ARG     mysql_database=icinga
ARG     mysql_root_pass=password
ARG     mysql_icinga_user=icinga
ARG     mysql_icinga_pass=icingapass
ARG     mysql_webdatabase=icingaweb2
ARG     mysql_icingaweb_user=icingaweb
ARG     mysql_icingaweb_pass=icingawebpass
ARG	icingaweb_admin_user=iadmin
ARG	icingaweb_admin_pass=ipass
ARG     mysql_dirdatabase=director
ARG     mysql_diruser=director
ARG     mysql_dirpassword=directorpass

ENV	MYSQL_ROOT_PASSWORD=$mysql_root_pass
ENV	MYSQL_DATABASE=$mysql_database
ENV	MYSQL_USER=$mysql_icinga_user
ENV	MYSQL_PASSWORD=$mysql_icinga_pass
ENV     MYSQL_WEBDATABASE=$mysql_webdatabase
ENV	MYSQL_WEBUSER=$mysql_icingaweb_user
ENV	MYSQL_WEBPASSWORD=$mysql_icingaweb_pass
ENV	ICINGAWEB_ADMIN_USER=$icingaweb_admin_user
ENV	ICINGAWEB_ADMIN_PASS=$icingaweb_admin_pass
ENV     MYSQL_DIRDATABASE=$mysql_dirdatabase
ENV     MYSQL_DIRUSER=$mysql_diruser
ENV     MYSQL_DIRPASSWORD=$mysql_dirpassword

RUN	apt -qq update \
	&& apt install -y --no-install-recommends --no-install-suggests curl gnupg ca-certificates \
	&& curl -s https://packages.icinga.com/icinga.key | apt-key add - \
	&&. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then DIST="${UBUNTU_CODENAME}"; else DIST="$(lsb_release -c| awk '{print $2}')"; fi; \
	echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" > \
	/etc/apt/sources.list.d/${DIST}-icinga.list \
	&& echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" >> \
	/etc/apt/sources.list.d/${DIST}-icinga.list \
	&& apt-mark hold mariadb* mysql*\
	&& apt -qq update \
	&& apt upgrade -y

RUN     DEBIAN_FRONTEND=noninteractive apt -qq update && DEBIAN_FRONTEND=noninteractive apt install -y icinga2-ido-mysql icingaweb2 php

RUN     sed -i 's/exec "$@"/echo \"not running $@\"/' /usr/local/bin/docker-entrypoint.sh

RUN	cat /usr/share/icinga2-ido-mysql/schema/mysql.sql \
	> /docker-entrypoint-initdb.d/setup.sql

RUN	echo "CREATE DATABASE $MYSQL_WEBDATABASE;\n\
	GRANT ALL ON $MYSQL_WEBDATABASE.* TO $MYSQL_WEBUSER IDENTIFIED BY \"$MYSQL_WEBPASSWORD\";\n"\
	> /docker-entrypoint-initdb.d/websetup.sql

RUN 	echo "USE $MYSQL_WEBDATABASE;\n" > /docker-entrypoint-initdb.d/websetup2.sql \
	&& cat /usr/share/icingaweb2/etc/schema/mysql.schema.sql \
	>> /docker-entrypoint-initdb.d/websetup2.sql

RUN	HASH=$(php -r "echo password_hash(\"$ICINGAWEB_ADMIN_PASS\", PASSWORD_DEFAULT);") \
	&& echo -n "USE $MYSQL_WEBDATABASE;\n\
        INSERT INTO icingaweb_user (name, active, password_hash) VALUES (\"$ICINGAWEB_ADMIN_USER\", 1, '" >> /docker-entrypoint-initdb.d/websetup3.sql \
	&& echo -n $HASH >> /docker-entrypoint-initdb.d/websetup3.sql \
	&& echo "');" >> /docker-entrypoint-initdb.d/websetup3.sql

RUN     echo "CREATE DATABASE $MYSQL_DIRDATABASE CHARACTER SET utf8;\n\
        GRANT ALL ON $MYSQL_DIRDATABASE.* TO $MYSQL_DIRUSER IDENTIFIED BY \"$MYSQL_DIRPASSWORD\";\n"\
        > /docker-entrypoint-initdb.d/websetup4.sql

RUN	/usr/local/bin/docker-entrypoint.sh mysqld --datadir=/initialized-db --aria-log-dir-path=/initialized-db

RUN	cp /docker-entrypoint-initdb.d/websetup3.sql /initialized-db

FROM	mariadb:10.4

COPY	--from=builder /initialized-db /var/lib/mysql
