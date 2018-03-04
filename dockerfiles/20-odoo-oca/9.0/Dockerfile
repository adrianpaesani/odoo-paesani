FROM apaesani/odoo-base:9.0
MAINTAINER Adrian Paesani <adrian.paesani@gmail.com>

USER root

# Get repositories & add files
WORKDIR ${RESOURCES}
COPY ./oca-sources.txt ./
RUN mkdir -p ${EXTRA_ADDONS}
RUN python get_addons.py ${EXTRA_ADDONS} --depth=1 --sources_file=oca-sources.txt
WORKDIR /

USER odoo
