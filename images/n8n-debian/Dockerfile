#python更新至3.11，nodejs版本不能大于22
#FROM nikolaik/python-nodejs:python3.11-nodejs22
FROM nikolaik/python-nodejs:python3.11-nodejs20

# changing user `pn` to `node`
RUN usermod --login node --move-home --home /home/node pn
RUN groupmod --new-name node pn

ARG N8N_VERSION

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

RUN \
	apt-get update && \
	apt-get -y install graphicsmagick gosu git

RUN apt-get update && apt-get install -y \
    python3-dev \
    gcc \
    g++ \
    tini \
    wget \
    curl \
    #linux-headers-$(uname -r) \
    build-essential && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 2. 下载、编译并安装 TA-Lib 的 C 库
RUN mkdir -p /tmp/ta-lib && \
    cd /tmp/ta-lib && \
    wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz && \
    tar -xzf ta-lib-0.4.0-src.tar.gz && \
    cd ta-lib/ && \
    ./configure --prefix=/usr/local && \
    make && \
    make install && \
    cd / && \
    rm -rf /tmp/ta-lib

# 更新库路径
RUN ldconfig
    
# Install python requirements
RUN mkdir /requirements
COPY requirements.txt /requirements/requirements.txt
RUN python -m pip install --upgrade pip setuptools wheel
RUN pip install -r /requirements/requirements.txt

# Set a custom user to not have n8n run as root
USER root

RUN npm_config_user=root npm install -g full-icu n8n@${N8N_VERSION}

# Install n8n-nodes-python module
#RUN cd /usr/lib/node_modules/n8n && npm install n8n-nodes-python
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-python
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-changed
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-updates
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-influxdb
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-tavily


ENV NODE_ICU_DATA /usr/lib/node_modules/full-icu

WORKDIR /data

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5678/tcp
