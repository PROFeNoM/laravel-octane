FROM ubuntu:22.04

LABEL maintainer="Taylor Otwell"

ARG WWWGROUP
ARG NODE_VERSION=20
ARG POSTGRES_VERSION=15

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC
ENV SUPERVISOR_PHP_COMMAND="/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan serve --host=0.0.0.0 --port=80"
ENV SUPERVISOR_PHP_USER="sail"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && mkdir -p /etc/apt/keyrings \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 dnsutils librsvg2-bin fswatch ffmpeg nano  \
    && curl -sS 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c' | gpg --dearmor | tee /etc/apt/keyrings/ppa_ondrej_php.gpg > /dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php8.3-cli php8.3-dev \
       php8.3-pgsql php8.3-sqlite3 php8.3-gd \
       php8.3-curl \
       php8.3-imap php8.3-mysql php8.3-mbstring \
       php8.3-xml php8.3-zip php8.3-bcmath php8.3-soap \
       php8.3-intl php8.3-readline \
       php8.3-ldap \
       php8.3-msgpack php8.3-igbinary php8.3-redis php8.3-swoole \
       php8.3-memcached php8.3-pcov php8.3-imagick php8.3-xdebug \
    && curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" > /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && npm install -g pnpm \
    && npm install -g bun \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /etc/apt/keyrings/yarn.gpg >/dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && curl -sS https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /etc/apt/keyrings/pgdg.gpg >/dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update \
    && apt-get install -y yarn \
    && apt-get install -y mysql-client \
    && apt-get install -y postgresql-client-$POSTGRES_VERSION \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.3

RUN groupadd --force -g $WWWGROUP sail
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 sail

RUN apt-get update && apt-get install -y php8.3-opcache
RUN PHP_INI_DIR=$(php -i | grep 'Scan this dir for additional .ini files' | awk '{print $NF}') \
    && echo "opcache.enable=1" > $PHP_INI_DIR/10-opcache.ini \
    && echo "opcache.enable_cli=1" >> $PHP_INI_DIR/10-opcache.ini \
    && echo "opcache.memory_consumption=128" >> $PHP_INI_DIR/10-opcache.ini \
    && echo "opcache.interned_strings_buffer=8" >> $PHP_INI_DIR/10-opcache.ini \
    && echo "opcache.max_accelerated_files=4000" >> $PHP_INI_DIR/10-opcache.ini \
    && echo "opcache.revalidate_freq=2" >> $PHP_INI_DIR/10-opcache.ini \
    && echo "opcache.fast_shutdown=1" >> $PHP_INI_DIR/10-opcache.ini

#RUN curl -LO https://github.com/DataDog/dd-trace-php/releases/latest/download/datadog-setup.php
#RUN curl -LO https://output.circle-artifacts.com/output/job/ce624919-3ea1-4414-9a21-a1495f62878a/artifacts/0/datadog-setup.php
#RUN curl -LO https://output.circle-artifacts.com/output/job/c96a8e7e-7fd6-4366-b00d-2358c1813e48/artifacts/0/datadog-setup.php
#RUN curl -LO https://output.circle-artifacts.com/output/job/4f76a8fb-30d4-48c6-b42a-25a088946ebd/artifacts/0/datadog-setup.php
#RUN curl -LO https://output.circle-artifacts.com/output/job/530c3ea1-337f-4ff2-b78a-d2daf2dc7357/artifacts/0/datadog-setup.php
RUN curl -LO https://output.circle-artifacts.com/output/job/f760d045-cd51-4ac5-951b-8edc6ea05637/artifacts/0/datadog-setup.php
RUN php datadog-setup.php --php-bin=all --enable-profiling

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/8.3/cli/conf.d/99-sail.ini
RUN chmod +x /usr/local/bin/start-container

EXPOSE 8000

ENTRYPOINT ["start-container"]
