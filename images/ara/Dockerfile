FROM quay.io/centos/centos:stream9

RUN dnf update -y \
    && dnf install -y \
      which \
      python3-pip \
      python3-pip-wheel \
      postgresql \
      libpq \
      mariadb-connector-c \
    && dnf clean all

ARG DEV_DEPENDENCIES="gcc python3-devel postgresql-devel mariadb-connector-c-devel"
RUN dnf install -y ${DEV_DEPENDENCIES}  \
    && python3 -m pip install --no-cache-dir \
      "ara[server,postgresql,mysql]" \
      gunicorn \
    && dnf remove -y ${DEV_DEPENDENCIES} \
    && dnf autoremove -y \
    && dnf clean all \
    && python3 -m pip cache purge

ENV ARA_BASE_DIR=/opt/ara

EXPOSE 8000

CMD ["/bin/bash", "-c", "/usr/local/bin/ara-manage migrate && python3 -m gunicorn --workers=4 --access-logfile - --bind 0.0.0.0:8000 ara.server.wsgi"]
