# Update the NODE_VERSION arg in docker-compose.yml to pick a Node version: 18, 16, 14
ARG NODE_VERSION=18
FROM mcr.microsoft.com/devcontainers/javascript-node:${NODE_VERSION}

# Set SHELL option -o pipefail
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# VARIANT can be either 'hugo' for the standard version or 'hugo_extended' for the extended version.
ARG VARIANT=hugo
# VERSION can be either 'latest' or a specific version number
ARG VERSION=latest

# Download Hugo
RUN apt-get update && apt-get install -y gosu ca-certificates openssl git curl --no-install-recommends && \
    rm -rf /var/lib/apt/lists/* && \
    case ${VERSION} in \
    latest) \
    export VERSION=$(curl -s https://api.github.com/repos/gohugoio/hugo/releases/latest | grep "tag_name" | awk '{print substr($2, 3, length($2)-4)}') ;;\
    esac && \
    echo ${VERSION} && \
    case $(uname -m) in \
    aarch64) \
    export ARCH=ARM64 ;; \
    *) \
    export ARCH=64bit ;; \
    esac && \
    echo ${ARCH} && \
    curl -L -o "${VERSION}.tar.gz" "https://github.com/gohugoio/hugo/releases/download/v${VERSION}/${VARIANT}_${VERSION}_Linux-${ARCH}.tar.gz" && \
    tar xf ${VERSION}.tar.gz && \
    mv hugo /usr/bin/hugo

# Hugo dev server port
EXPOSE 1313

# [Optional] Uncomment this section to install additional OS packages you may want.
#
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment if you want to install more global node packages
# RUN gosu node npm install -g <your-package-list-here>
# Install stylelint/postcss. If a project `package.json` exists, prefer installing devDependencies using `npm ci` into /usr/local so tools are available globally.
COPY package.json package-lock.json* /tmp/
# Make /usr/local writable by the 'node' user so `npm --prefix /usr/local` can write files there
RUN mkdir -p /usr/local && chown -R node:node /usr/local
WORKDIR /tmp
RUN if [ -f /tmp/package.json ]; then \
      mkdir -p /usr/local/bin && \
      if [ -f package-lock.json ]; then \
        # Install into /tmp first to avoid npm looking for package.json under /usr/local
        gosu node npm ci --no-audit --no-fund --prefix /tmp; \
      else \
        gosu node npm install --no-audit --no-fund --prefix /tmp; \
      fi && \
      # Move installed modules and executables into /usr/local
      cp -a /tmp/node_modules /usr/local/ || true && \
      if [ -d /tmp/node_modules/.bin ]; then cp -a /tmp/node_modules/.bin /usr/local/bin/ || true; fi && \
      chown -R node:node /usr/local && rm -rf /tmp/node_modules || true; \
    else \
      gosu node npm install -g stylelint postcss stylelint-config-standard-scss; \
    fi
