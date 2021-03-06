FROM nginx:1.15.7-alpine

MAINTAINER CodeAnti <1428491590@qq.com>

RUN echo @testing http://nl.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories && \
    echo /etc/apk/respositories && \
    apk update && \
    apk add --no-cache bash \
    openssh-client \
    openssl \
    wget \
    supervisor \
    curl \
    libcurl \
    git \
    python \
    py-pip \
    openssl-dev \
    ca-certificates \
    dialog \
    gcc \
    libxslt-dev \
    freetype-dev

# Copy our nginx config
RUN rm -Rf /etc/nginx/nginx.conf
ADD conf/nginx.conf /etc/nginx/nginx.conf

# Add nginx dir
RUN mkdir -p /data/nginx/conf

# Add Site nginx config
# ADD conf/nginx-site.conf /etc/nginx/conf/

# Add supervisord conf
RUN mkdir -p /etc/supervisor/conf.d
ADD supervisor/supervisord.conf /etc/supervisord.conf
ADD supervisor/conf.d/ /etc/supervisor/conf.d/

# Modify Timezone
ENV TZ Asia/Shanghai
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add Vim
RUN apk add --no-cache vim && \
    rm -f /usr/bin/vi && \
    cp /usr/bin/vim /usr/bin/vi

# Add Cron
COPY cron/schedule /etc/cron.d/

# Modify Cron Configs
RUN crontab /etc/cron.d/schedule && \
    mkdir -p /var/log/cron && \
    touch /var/log/cron/cron.log && \
    touch /var/log/cron.log && \
    touch /tmp/cron.log && \
    chown nginx.nginx /tmp/cron.log /var/log/cron/cron.log

# Optimize bash
ADD bash/.bashrc /root/.bashrc
ADD bash/.inputrc /root/.inputrc

# Add Scripts
ADD scripts/start.sh /start.sh

# Add Start Script Priviliages
RUN chmod +x /start.sh

WORKDIR /etc/nginx/conf

EXPOSE 80

VOLUME ["/etc/nginx/conf"]

CMD ["/start.sh"]