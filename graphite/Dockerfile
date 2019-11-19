FROM ubuntu:bionic as builder
MAINTAINER Alexander Vlasov <vlasov@gmail.com>

ARG	secret_key="UNSAFE_DEFAULT"

ENV	SECRET_KEY=$secret_key


RUN	apt -qq update \
        && apt upgrade -y \
	&& apt install -y --no-install-recommends --no-install-suggests python-dev python-pip libcairo2-dev libffi-dev build-essential gunicorn

RUN	PYTHONPATH="/opt/graphite/lib/:/opt/graphite/webapp/" \ 
	pip install --no-binary=:all: https://github.com/graphite-project/whisper/tarball/master \
	&& pip install --no-binary=:all: https://github.com/graphite-project/carbon/tarball/master \
	&& pip install --no-binary=:all: https://github.com/graphite-project/graphite-web/tarball/master

RUN	echo -n "SECRET_KEY = '" > /opt/graphite/webapp/graphite/local_settings.py \
	&& echo -n $SECRET_KEY >> /opt/graphite/webapp/graphite/local_settings.py \
	&& echo "'" >> /opt/graphite/webapp/graphite/local_settings.py

RUN	PYTHONPATH=/opt/graphite/webapp django-admin.py migrate --settings=graphite.settings

RUN	chown www-data:www-data /opt/graphite/storage/graphite.db

RUN	cd /opt/graphite/conf \
	&& cp carbon.conf.example carbon.conf \
	&& cp storage-schemas.conf.example storage-schemas.conf

RUN	ln -sf /dev/stderr /var/log/gunicorn.log

CMD	PYTHONPATH="/opt/graphite/lib/:/opt/graphite/webapp/" /opt/graphite/bin/carbon-cache.py start && PYTHONPATH=/opt/graphite/webapp gunicorn --log-level info --workers=4 --bind=:8080 --log-file=/var/log/gunicorn.log --preload --pythonpath=/opt/graphite/webapp/graphite wsgi
