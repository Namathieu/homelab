FROM ubuntu:24.04

ARG file_name=ZenithProxy-launcher-linux-amd64.zip

WORKDIR /app

RUN apt-get update && apt-get install -y wget unzip ca-certificates && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN wget -O /tmp/$file_name https://github.com/rfresh2/ZenithProxy/releases/download/launcher-v3/$file_name && \
    unzip /tmp/$file_name -d /app && \
    rm /tmp/$file_name && \
    chmod +x /app/* && \
    ln -sf /app/ZenithProxy-launcher-linux-amd64 /app/launch

ENTRYPOINT ["/app/launch"]