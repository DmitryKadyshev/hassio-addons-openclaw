ARG BUILD_FROM=ghcr.io/home-assistant/base-debian:bookworm
FROM ${BUILD_FROM}

# Base image is Debian Bookworm (glibc). This avoids musl-related native module issues
# that occur on Alpine (e.g. clipboard, node-llama-cpp).

# Install base packages (without nodejs/npm - we'll get Node 24 from NodeSource)
# build-essential provides gcc/cc needed by Homebrew for OpenClaw's brew-managed dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    bash git openssh-client ca-certificates tzdata jq curl tar xz-utils file python3 \
    cmake nginx gnupg build-essential sudo vim nano fd-find ripgrep rsync bat less openssl \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Node.js 24 LTS from NodeSource.
# OpenClaw supports Node >=24.15.0. Node 24 does not publish an official Linux armv7 binary,
# so this add-on is intentionally limited to the currently supported amd64/arm64 platforms.
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
    | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && chmod 644 /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_24.x nodistro main" \
    > /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends nodejs \
    && NODE_VERSION="$(node -v)" \
    && echo "$NODE_VERSION" \
    && echo "$NODE_VERSION" | grep -E '^v24\.' \
    && rm -f /etc/apt/sources.list.d/nodesource.list /etc/apt/keyrings/nodesource.gpg \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install ttyd (web terminal) for the supported architectures.
ARG TARGETARCH
RUN ARCH=$(echo ${TARGETARCH:-$(dpkg --print-architecture)} | sed 's|arm64|aarch64|;s|amd64|x86_64|') \
    && echo "Downloading ttyd for arch: ${ARCH}" \
    && curl -fsSL "https://github.com/tsl0922/ttyd/releases/download/1.7.7/ttyd.${ARCH}" -o /usr/local/bin/ttyd \
    && chmod +x /usr/local/bin/ttyd

RUN node -v && npm -v

# Install pnpm globally (required by some OpenClaw skills like clawhub)
RUN npm install -g pnpm && pnpm -v

# Install Chromium for website automation tasks
RUN apt-get update && apt-get install -y --no-install-recommends chromium chromium-driver \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Homebrew (Linuxbrew) for OpenClaw skill dependencies
ENV HOMEBREW_NO_AUTO_UPDATE=1 \
    HOMEBREW_NO_INSTALL_CLEANUP=1 \
    HOMEBREW_NO_ANALYTICS=1

RUN useradd -m -s /bin/bash linuxbrew \
    && mkdir -p /home/linuxbrew/.linuxbrew \
    && chown -R linuxbrew:linuxbrew /home/linuxbrew

USER linuxbrew
RUN /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" || \
    (echo "WARNING: Homebrew installation failed (likely unsupported CPU - requires SSSE3). Some skills may not work." && exit 0)
RUN if [ -d /home/linuxbrew/.linuxbrew/Homebrew ]; then \
    cd /home/linuxbrew/.linuxbrew/Homebrew && \
    git config --global --add safe.directory /home/linuxbrew/.linuxbrew/Homebrew && \
    /home/linuxbrew/.linuxbrew/bin/brew update --force || true; \
    fi
USER root
ENV PATH="/usr/local/bin:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:${PATH}"

COPY brew-wrapper.sh /usr/local/bin/brew
RUN chmod +x /usr/local/bin/brew

USER linuxbrew
RUN if [ -x /home/linuxbrew/.linuxbrew/bin/brew ]; then \
    /home/linuxbrew/.linuxbrew/bin/brew --version && /home/linuxbrew/.linuxbrew/bin/brew install gcc || true; \
    else echo "Skipping gcc installation - Homebrew not available"; fi
USER root

# Install OpenClaw and its bundled native/MCP dependencies.
RUN npm config set fund false && npm config set audit false \
    && npm install -g openclaw@2026.8.2 node-llama-cpp@3.18.1 mcporter@0.12.3

RUN tee -a /etc/bash.bashrc <<'EOF'
alias bat="batcat"
alias fd="fdfind"
alias ls="ls --color=auto"
alias grep="grep --color=auto"
alias egrep="grep -E --color=auto"
alias fgrep="grep -F --color=auto"
alias diff="diff --color=auto"
alias ip="ip --color=auto"
if [ -z "${LS_COLORS+x}" ]; then
  export LS_COLORS="di=1;34:ln=1;36:so=1;35:pi=33:ex=1;32:bd=1;33:cd=1;33:su=1;31:sg=1;31:tw=1;34:ow=1;34"
fi
EOF

ARG BUILD_VERSION
ENV ADDON_VERSION=${BUILD_VERSION}

COPY run.sh /run.sh
COPY oc_config_helper.py /oc_config_helper.py
COPY render_nginx.py /render_nginx.py
COPY oc-cleanup.sh /usr/local/bin/oc-cleanup
COPY oc-gateway /usr/local/bin/oc-gateway
COPY oc-health.sh /usr/local/bin/oc-health
COPY oc-config.sh /usr/local/bin/oc-config
COPY openclaw-proxy-shim.cjs /usr/local/lib/openclaw-proxy-shim.cjs
COPY nginx.conf.tpl /etc/nginx/nginx.conf.tpl
COPY landing.html.tpl /etc/nginx/landing.html.tpl
RUN chmod +x /run.sh /oc_config_helper.py /render_nginx.py /usr/local/bin/oc-cleanup /usr/local/bin/oc-gateway \
    /usr/local/bin/oc-health /usr/local/bin/oc-config \
    && mkdir -p /run/nginx

CMD [ "/run.sh" ]
