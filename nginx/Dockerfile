FROM ubuntu:bionic as builder
MAINTAINER Alexander Vlasov <vlasov@gmail.com>

RUN	apt -qq update \
	&& apt upgrade -y \
	&& DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends --no-install-suggests nginx

RUN	rm -rf /etc/nginx/sites-enabled

RUN	ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE	80 443

CMD ["/usr/sbin/nginx","-g","daemon off;"]
