#更新为python3.11 
FROM nikolaik/python-nodejs:python3.11-nodejs22-alpine

# changing user `pn` to `node`
RUN deluser pn && rm -r /home/pn # delete: user + group
RUN addgroup -g 1000 -S node && adduser -u 1000 -S node -G node -s /bin/sh # create: user + group

ARG N8N_VERSION

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

# Update everything and install needed dependencies
RUN apk add --update graphicsmagick tzdata git tini su-exec

RUN apk add --no-cache \
    python3-dev \
    py3-pip \
    gcc \
    g++ \
    musl-dev \
    linux-headers \
    build-base
    
# # Set a custom user to not have n8n run as root
USER root

# Install n8n and the also temporary all the packages
# it needs to build it correctly.
RUN apk --update add --virtual build-dependencies build-base ca-certificates && \
	npm_config_user=root npm install -g full-icu n8n@${N8N_VERSION} && \
	apk del build-dependencies \
	&& rm -rf /root /tmp/* /var/cache/apk/* && mkdir /root;

# Install n8n-nodes-python module
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-python

# Install fonts
RUN apk --no-cache add --virtual fonts msttcorefonts-installer fontconfig && \
	update-ms-fonts && \
	fc-cache -f && \
	apk del fonts && \
	find  /usr/share/fonts/truetype/msttcorefonts/ -type l -exec unlink {} \; \
	&& rm -rf /root /tmp/* /var/cache/apk/* && mkdir /root

# 创建 Python 辅助脚本，方便 ExecuteCommand 节点调用
RUN echo '#!/bin/sh\nPYTHONPATH=/usr/local/lib/python3.11/site-packages:/home/pyodide/lib/python3.11/site-packages exec /usr/local/bin/python3 "$@"' > /usr/local/bin/n8n-python && \
    chmod +x /usr/local/bin/n8n-python

RUN pip install --no-cache-dir markitdown
# Install python requirements
RUN mkdir /requirements
COPY requirements.txt /requirements/requirements.txt
RUN python -m pip install --upgrade pip setuptools wheel
#RUN pip install -r /requirements/requirements.txt
RUN pip install --no-cache-dir -r /requirements/requirements.txt
ENV NODE_ICU_DATA /usr/local/lib/node_modules/full-icu

# 设置 PYTHONPATH 环境变量，确保 Python 能找到所有已安装的包
ENV PYTHONPATH=/usr/local/lib/python3.11/site-packages:/home/pyodide/lib/python3.11/site-packages

WORKDIR /data

# 直接在 Dockerfile 中创建 entrypoint 脚本
RUN echo '#!/bin/sh\n\
set -e\n\
\n\
# 设置 PYTHONPATH 环境变量\n\
export PYTHONPATH=/usr/local/lib/python3.11/site-packages:/home/pyodide/lib/python3.11/site-packages:$PYTHONPATH\n\
\n\
# 如果命令以 n8n 开头，则使用 node 用户运行\n\
if [ "${1:0:4}" = "n8n " ]; then\n\
  exec su-exec node "$@"\n\
else\n\
  exec "$@"\n\
fi' > /docker-entrypoint.sh && \
    chmod +x /docker-entrypoint.sh

ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

EXPOSE 5678/tcp
