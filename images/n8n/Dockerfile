#更新为python3.11 ,18.7<nodejs<=22
FROM nikolaik/python-nodejs:python3.11-nodejs22-alpine

# changing user `pn` to `node`
RUN deluser pn && rm -r /home/pn # delete: user + group
RUN addgroup -g 1000 -S node && adduser -u 1000 -S node -G node -s /bin/sh # create: user + group

ARG N8N_VERSION

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

# 使用 apt-get 替代 apk
RUN apt-get update && apt-get install -y graphicsmagick tzdata git tini su-exec

# # Set a custom user to not have n8n run as root
USER root

# 使用 apt-get 安装构建依赖
RUN apt-get update && apt-get install -y build-essential ca-certificates && \
	npm_config_user=root npm install -g full-icu n8n@${N8N_VERSION} && \
	apt-get purge -y build-essential && apt-get autoremove -y && \
	rm -rf /root /tmp/* /var/cache/apk/* && mkdir /root

# Install n8n-nodes-python module
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-python
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-changed
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-updates
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-influxdb

# 使用 apt-get 安装字体
RUN apt-get update && apt-get install -y fonts-liberation fontconfig && \
	fc-cache -f && \
	apt-get purge -y fonts-liberation && apt-get autoremove -y && \
	rm -rf /root /tmp/* /var/lib/apt/lists/* && mkdir /root

# Install python requirements
RUN mkdir /requirements
COPY requirements.txt /requirements/requirements.txt
RUN python -m pip install --upgrade pip setuptools wheel
RUN pip install --no-cache-dir -r /requirements/requirements.txt
ENV NODE_ICU_DATA /usr/local/lib/node_modules/full-icu

WORKDIR /data

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

EXPOSE 5678/tcp
