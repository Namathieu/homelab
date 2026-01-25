FROM ubuntu:24.04

ARG file_name=ZenithProxy-launcher-linux-amd64.zip

# Put the launcher somewhere that won't be overwritten by a volume mount
WORKDIR /opt/zenith

RUN apt-get update && apt-get install -y wget unzip ca-certificates && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN wget -O /tmp/$file_name https://github.com/rfresh2/ZenithProxy/releases/download/launcher-v3/$file_name && \
    unzip /tmp/$file_name -d /opt/zenith && \
    rm /tmp/$file_name && \
    chmod +x /opt/zenith/* && \
    ln -sf /opt/zenith/ZenithProxy-launcher-linux-amd64 /usr/local/bin/zenithproxy

# The config/data lives here (this is what you'll mount per account)
WORKDIR /config

ENTRYPOINT ["/usr/local/bin/zenithproxy"]