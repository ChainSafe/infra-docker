ARG FOREST_VERSION=v0.29.0-fat

FROM ghcr.io/chainsafe/forest:${FOREST_VERSION}
ENV DEBIAN_FRONTEND="noninteractive"

# Install binary dependencies
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
      bc \
      curl \
      jq \
      python3 \
      python3-pip \
      unzip  \
    && rm -rf /var/lib/apt/lists/*

# Install awscli
WORKDIR /opt
RUN \
  curl -sL "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" \
  && unzip -qq awscliv2.zip \
  && rm -f awscliv2.zip \
  && ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update > /dev/null \
  && rm -rf \
    /usr/local/aws-cli/v2/current/dist/aws_completer \
    /usr/local/aws-cli/v2/current/dist/awscli/data/ac.index \
    /usr/local/aws-cli/v2/current/dist/awscli/examples \
  && find /usr/local/aws-cli/v2/current/dist/awscli/data -name 'completions-1*.json' -delete \
  && find /usr/local/aws-cli/v2/current/dist/awscli/botocore/data -name examples-1.json -delete

# Install Pika
RUN pip3 install pika --break-system-packages

# Basic verification of dynamically linked dependencies
RUN forest -V \
    && forest-cli -V \
    && forest-tool -V \
    && aws --version \
    && jq -V \
    && python3 -V

ENTRYPOINT ["/bin/bash"]
