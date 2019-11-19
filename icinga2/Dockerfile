FROM ubuntu:bionic
MAINTAINER Alexander Vlasov <vlasov@gmail.com>

ARG	icinga_host=icinga
ARG	mysql_host=mariadb
ARG	mysql_database=icinga
ARG	mysql_icinga_user=icinga
ARG	mysql_icinga_pass=icingapass
ARG	api_user=api
ARG	api_password=apipass
ARG	dir_user=dir
ARG	dir_password=dirpass
ARG	graphite_host=graphite

ENV	ICINGA_HOST=$icinga_host
ENV	MYSQL_HOST=$mysql_host
ENV	MYSQL_DATABASE=$mysql_database
ENV	MYSQL_ICINGA_USER=$mysql_icinga_user
ENV	MYSQL_ICINGA_PASS=$mysql_icinga_pass
ENV	API_USER=$api_user
ENV	API_PASSWORD=$api_password
ENV	DIR_USER=$dir_user
ENV	DIR_PASSWORD=$dir_password
ENV	GRAPHITE_HOST=$graphite_host


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

RUN	DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends --no-install-suggests icinga2 nagios-plugins monitoring-plugins icinga2-ido-mysql

RUN	apt clean

RUN	echo "library \"db_ido_mysql\" \n\
        object IdoMysqlConnection \"ido-mysql\" { \n\
        user = \"$MYSQL_ICINGA_USER\" \n\
        password = \"$MYSQL_ICINGA_PASS\" \n\
        host = \"$MYSQL_HOST\" \n\
        database = \"$MYSQL_DATABASE\" \n\
	} \n"\
	> /etc/icinga2/features-available/ido-mysql.conf

RUN	echo "object ApiUser \"$API_USER\" { \n\
	password = \"$API_PASSWORD\"\n\
	permissions = [ \"status/query\", \"actions/*\", \"objects/modify//*\", \"objects/query/*\" ]\n\
	}\n\
	object ApiUser \"$DIR_USER\" { \n\
        password = \"$DIR_PASSWORD\"\n\
        permissions = [ \"*\" ]\n\
        }\n"\
	> /etc/icinga2/conf.d/api-users.conf

RUN	icinga2 feature enable graphite

RUN     echo "library \"perfdata\" \n\
	\n\
	object GraphiteWriter \"graphite\" {\n\
	host = \"$GRAPHITE_HOST\"\n\
	port = 2003\n\ 
	enable_send_thresholds = true\n\
	}\n"\
	> /etc/icinga2/features-enabled/graphite.conf

RUN	mkdir -p /run/icinga2 \
	&& chown -R nagios:nagios /run/icinga2

RUN	icinga2 feature enable ido-mysql

RUN	sed -i "s/severity = \"information\"/severity = \"critical\"/" /etc/icinga2/features-available/mainlog.conf

RUN	echo "" > /etc/icinga2/conf.d/hosts.conf

# Workaround for hostname during build
RUN	echo "$ICINGA_HOST" > /etc/hostname && echo $(grep $(hostname) /etc/hosts | cut -f1) $ICINGA_HOST > /etc/hosts&& icinga2 api setup --cn $ICINGA_HOST

RUN	ln -sf /dev/stderr /var/log/icinga2/startup.log

EXPOSE	5665

CMD 	/etc/init.d/icinga2 foreground > /dev/null
	