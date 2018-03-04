FROM ubuntu:xenial
MAINTAINER Adrian Paesani <adrian.paesani@gmail.com>

# segun odoo esta version sigue siendo la recomendable https://github.com/odoo/odoo/wiki/Wkhtmltopdf, la 12.2.1 y las podemos encontrar acá https://wkhtmltopdf.org/downloads.html

ENV WKHTMLTOX_X64 https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.2.1/wkhtmltox-0.12.2.1_linux-trusty-amd64.deb

# esta es la de odoo pero es para jessie y no anduvo
# ENV WKHTMLTOX_X64 http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb
# depreciado, no existe mas
# ENV WKHTMLTOX_X64 http://download.gna.org/wkhtmltopdf/0.12/0.12.1/wkhtmltox-0.12.1_linux-trusty-amd64.deb

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            node-less \
            node-clean-css \
            python-pyinotify \
            python-renderpm
            # not anymore on ubuntu 16.04
            # python-support
RUN set -x; \
        curl -o wkhtmltox.deb -SL ${WKHTMLTOX_X64} \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f --no-install-recommends \
        && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install Odoo
ENV ODOO_VERSION 9.0
# revertimos por error con Unknown path: /base/rng/view.rng y repositorio enterprise no en ultima version
ENV ODOO_RELEASE 20180216

ENV PYTHONIOENCODING utf-8

ENV TERM xterm

#Install Odoo dependencies
RUN apt-get update -q && apt-get upgrade -q && \
    apt-get install --allow-unauthenticated -q \
    wget

RUN apt-get update -q && apt-get upgrade -q && \
    apt-get install --allow-unauthenticated -q \
    fontconfig \
    libevent-dev \
    libjpeg-dev \
    libldap2-dev \
    libsasl2-dev \
    libx11-6 \
    libxext6 \
    libxml2-dev \
    libxrender1 \
    libxslt-dev \
    nano \
    npm \
    postgresql-server-dev-9.4 \
    python \
    python-dev \
    xfonts-75dpi \
    xfonts-base

# Install nodejs
RUN ln -s /usr/bin/nodejs /usr/bin/node && \
    npm install -g less less-plugin-clean-css

# Install pip
RUN cd /tmp && \
    wget -q https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py

# Download and install odoo requirements from github.com/odoo/odoo/requirements.txt
RUN cd /tmp && wget -q https://raw.githubusercontent.com/odoo/odoo/9.0/requirements.txt && pip install -r requirements.txt

# Add user
RUN adduser --home=/opt/odoo --disabled-password --gecos "" --shell=/bin/bash odoo
RUN echo 'root:odoo' |chpasswd

# Add Logrotate file
ADD files/logrotate /etc/logrotate.d/odoo-server
RUN chmod 755 /etc/logrotate.d/odoo-server

# Add log directory
RUN mkdir -p /var/log/odoo && \
    chown odoo:root /var/log/odoo

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./openerp-server.conf /etc/odoo/
RUN chown odoo /etc/odoo/openerp-server.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV OPENERP_SERVER /etc/odoo/openerp-server.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["openerp-server"]
