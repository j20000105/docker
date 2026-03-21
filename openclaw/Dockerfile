FROM alpine/openclaw

USER root
RUN apt-get update && apt-get install -y curl vim chromium python3 python3-pip && \
    ln -sf /usr/bin/python3 /usr/bin/python && \
    ln -sf /usr/bin/pip3 /usr/bin/pip && \
    curl -LsSf https://astral.sh/uv/install.sh | env INSTALLER_NO_MODIFY_PATH=1 sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/ && \
    rm -rf /var/lib/apt/lists/* /root/.local && \
    mkdir -p /home/node/.openclaw && \
    chown -R node:node /home/node/.openclaw
USER node

RUN mkdir -p /home/node/.openclaw/.bilibili-cli && \
    ln -s /home/node/.openclaw/.bilibili-cli /home/node/.bilibili-cli
ENV NPM_CONFIG_PREFIX=/home/node/.openclaw/.npm-global
ENV PATH=$PATH:/home/node/.openclaw/bin:/home/node/.openclaw/.npm-global/bin
