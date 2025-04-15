#python更新至3.11，nodejs版本不能大于22
FROM nikolaik/python-nodejs:python3.11-nodejs22

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
    #linux-headers-$(uname -r) \
    build-essential && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
    
# 创建 Pyodide 目录结构
RUN mkdir -p /home/pyodide/lib/python3.11/site-packages

# Install python requirements
RUN mkdir /requirements
COPY requirements.txt /requirements/requirements.txt
RUN python -m pip install --upgrade pip setuptools wheel
RUN pip install -r /requirements/requirements.txt

# 创建符号链接，将系统 Python 包链接到 Pyodide 环境
RUN ln -s /usr/local/lib/python3.11/dist-packages/* /home/pyodide/lib/python3.11/site-packages/ || true

# 创建 Python 辅助脚本，方便 ExecuteCommand 节点调用
RUN echo '#!/bin/bash\nPYTHONPATH=/usr/local/lib/python3.11/dist-packages:/home/pyodide/lib/python3.11/site-packages:$PYTHONPATH exec /usr/bin/python3 "$@"' > /usr/local/bin/n8n-python && \
    chmod +x /usr/local/bin/n8n-python

# Set a custom user to not have n8n run as root
USER root

RUN npm_config_user=root npm install -g full-icu n8n@${N8N_VERSION}

# Install n8n-nodes-python module
#RUN cd /usr/lib/node_modules/n8n && npm install n8n-nodes-python
RUN cd /usr/local/lib/node_modules/n8n && npm install n8n-nodes-python

# 设置 PYTHONPATH 环境变量，确保 Python 能找到所有已安装的包
ENV PYTHONPATH=/usr/local/lib/python3.11/dist-packages:/home/pyodide/lib/python3.11/site-packages
ENV NODE_ICU_DATA /usr/lib/node_modules/full-icu

WORKDIR /data

COPY docker-entrypoint.sh /docker-entrypoint.sh
# 修改 entrypoint 脚本以设置 PYTHONPATH
RUN sed -i '1s/^/export PYTHONPATH=\/usr\/local\/lib\/python3.11\/dist-packages:\/home\/pyodide\/lib\/python3.11\/site-packages:$PYTHONPATH\n/' /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5678/tcp
