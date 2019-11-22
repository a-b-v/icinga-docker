# icinga-docker
Easy to run Icinga+graphite in docker environment.

Contains MariaDB, php-fpm (icingaweb2+icinga director), nginx, icinga2 and graphite Dockerfiles and docker-compose config.
Databases and host names, logins and passwords are configured trough args in docker-compose.yml

**Setup:** edit the arguments in docker-compose.yml

**Run:** execute "docker-compose up"
