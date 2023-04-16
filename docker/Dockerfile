FROM python:3.10-slim as builder
COPY package.json .npmrc pnpm-lock.yaml /tmp/build/
RUN set -x && \
  apt update -y && \
  apt install --no-install-recommends -y curl git && \
  curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
  apt update -y && \
  apt install --no-install-recommends nodejs -y && \
  npm i -g pnpm && \
  cd /tmp/build && \
  pnpm install --prod

FROM python:3.10-slim

ARG QL_MAINTAINER="whyour"
LABEL maintainer="${QL_MAINTAINER}"
ARG QL_URL=https://github.com/${QL_MAINTAINER}/qinglong.git
ARG QL_BRANCH=develop

ENV PNPM_HOME=/root/.local/share/pnpm \
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/.local/share/pnpm:/root/.local/share/pnpm/global/5/node_modules:$PNPM_HOME \
  NODE_PATH=/usr/local/bin:/usr/local/pnpm-global/5/node_modules:/usr/local/lib/node_modules:/root/.local/share/pnpm/global/5/node_modules \
  LANG=zh_CN.UTF-8 \
  SHELL=/bin/bash \
  PS1="\u@\h:\w \$ " \
  QL_DIR=/ql \
  QL_BRANCH=${QL_BRANCH}

RUN set -x && \
  sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list && \
  apt update && \
  apt upgrade -y && \
  apt install --no-install-recommends -y moreutils \
    git \
    curl \
    wget \
    nginx \
    jq && \
  curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
  apt update -y && \
  apt install --no-install-recommends -y nodejs && \
  apt clean && \
  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
  echo "Asia/Shanghai" >/etc/timezone && \
  git config --global user.email "qinglong@@users.noreply.github.com" && \
  git config --global user.name "qinglong" && \
  git config --global http.postBuffer 524288000 && \
  npm install -g pnpm && \
  pnpm add -g pm2 tsx && \
  rm -rf /root/.pnpm-store && \
  rm -rf /root/.local/share/pnpm/store && \
  rm -rf /root/.cache && \
  rm -rf /root/.npm

ARG SOURCE_COMMIT
RUN git clone -b ${QL_BRANCH} ${QL_URL} ${QL_DIR} && \
  cd ${QL_DIR} && \
  cp -f .env.example .env && \
  chmod 777 ${QL_DIR}/shell/*.sh && \
  chmod 777 ${QL_DIR}/docker/*.sh && \
  git clone -b ${QL_BRANCH} https://github.com/${QL_MAINTAINER}/qinglong-static.git /static && \
  mkdir -p ${QL_DIR}/static && \
  cp -rf /static/* ${QL_DIR}/static && \
  rm -rf /static

COPY --from=builder /tmp/build/node_modules/. /ql/node_modules/

WORKDIR ${QL_DIR}

ENTRYPOINT ["./docker/docker-entrypoint.sh"]
