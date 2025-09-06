ARG FOREST_VERSION=v0.29.0-fat
FROM golang:1.24-bookworm AS build-env
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# install dependencies
RUN apt-get update && \
    apt-get install --no-install-recommends -y build-essential clang-14 curl git ca-certificates
RUN update-ca-certificates
ENV CC=clang-14 CXX=clang++-14

# install Rust
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --no-modify-path --profile minimal
ENV PATH="/root/.cargo/bin:${PATH}"
ARG FOREST_BRANCH_VERSION="hm/fix-state-compute-for-null-epoch"
RUN git clone \
    --branch "${FOREST_BRANCH_VERSION}" \
    --single-branch  \
    https://github.com/ChainSafe/forest.git \
    /forest

WORKDIR /forest

# Install Forest. Move it out of the cache for the prod image.
RUN --mount=type=cache,sharing=private,target=/root/.cargo/registry \
    --mount=type=cache,sharing=private,target=/root/.rustup \
    --mount=type=cache,sharing=private,target=/forest/target \
    make install && \
    mkdir /forest_out && \
    cp /root/.cargo/bin/forest* /forest_out

FROM ubuntu:24.04 AS slim-image

ENV DEBIAN_FRONTEND="noninteractive"
# Install binary dependencies
RUN apt-get update && \
    apt-get install --no-install-recommends -y ca-certificates && \
    rm -rf /var/lib/apt/lists/*
RUN update-ca-certificates

# Copy forest daemon and cli binaries from the build-env
COPY --from=build-env /forest_out/* /usr/local/bin/

# Basic verification of dynamically linked dependencies
RUN forest -V && forest-cli -V && forest-tool -V && forest-wallet -V

ENTRYPOINT ["forest"]

# A fat image contains forest binaries and fil proof parameter files under $FIL_PROOFS_PARAMETER_CACHE
FROM slim-image AS fat-image

# Move FIL_PROOFS_PARAMETER_CACHE out of forest data dir since users always need to mount the data dir
ENV FIL_PROOFS_PARAMETER_CACHE="/var/tmp/filecoin-proof-parameters"

# Populate $FIL_PROOFS_PARAMETER_CACHE
RUN forest-tool fetch-params --keys

# Cache actor bundle in the image
ENV FOREST_ACTOR_BUNDLE_PATH="/var/tmp/forest_actor_bundle.car.zst"

# Populate $FOREST_ACTOR_BUNDLE_PATH
RUN forest-tool state-migration actor-bundle $FOREST_ACTOR_BUNDLE_PATH


# FIXME: revert back
# FROM ghcr.io/chainsafe/forest:${FOREST_VERSION}
ENV DEBIAN_FRONTEND="noninteractive"

# Install binary dependencies
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
      bc \
      curl \
      jq \
      python3 \
      python3-pip \
      unzip \
      wget  \
    && rm -rf /var/lib/apt/lists/*

# Install awscli
#WORKDIR /opt
#RUN \
#  curl -sL "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" \
#  && unzip -qq awscliv2.zip \
#  && rm -f awscliv2.zip \
#  && ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update > /dev/null \
#  && rm -rf \
#    /usr/local/aws-cli/v2/current/dist/aws_completer \
#    /usr/local/aws-cli/v2/current/dist/awscli/data/ac.index \
#    /usr/local/aws-cli/v2/current/dist/awscli/examples \
#  && find /usr/local/aws-cli/v2/current/dist/awscli/data -name 'completions-1*.json' -delete \
#  && find /usr/local/aws-cli/v2/current/dist/awscli/botocore/data -name examples-1.json -delete

# Install Python dependencies
COPY requirements.txt .
RUN pip3 install --no-cache-dir --break-system-packages -r requirements.txt \
    && rm -f requirements.txt

# Basic verification of dynamically linked dependencies
RUN forest -V \
    && forest-cli -V \
    && forest-tool -V \
    && aws --version \
    && jq -V \
    && python3 -V

ENTRYPOINT ["/bin/bash"]
