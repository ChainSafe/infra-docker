ARG AWX_EE_VERSION="24.6.1"
FROM quay.io/ansible/awx-ee:${AWX_EE_VERSION}

USER root
RUN python3 -m pip install \
        --upgrade --no-cache-dir  \
        ansible  \
        ara \
        pip \
    && python3 -m pip cache purge

USER 1000
