# syntax=docker/dockerfile:1

FROM debian:bookworm-slim@sha256:7c7b2c966bc9ee8cedfeef67e0e279108992c77681fa595db4a9d65c06ccc587

ARG UID=1000
ARG GID=1000

ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8 \
    HOME=/home/dev \
    SHELL=/bin/zsh

RUN rm -f /etc/dpkg/dpkg.cfg.d/docker \
    && apt-get update && apt-get install -y --no-install-recommends \
      bash \
      bat \
      build-essential \
      ca-certificates \
      curl \
      fd-find \
      fzf \
      git \
      jq \
      less \
      locales \
      man-db \
      passwd \
      pkg-config \
      psmisc \
      ripgrep \
      unzip \
      xz-utils \
      zsh \
    && sed -i 's/^# *\(en_US.UTF-8 UTF-8\)/\1/' /etc/locale.gen \
    && locale-gen \
    && ln -sf /usr/bin/batcat /usr/local/bin/bat \
    && ln -sf /usr/bin/fdfind /usr/local/bin/fd \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd --gid "${GID}" dev \
    && useradd --uid "${UID}" --gid "${GID}" --create-home --shell /bin/zsh dev \
    && install -d -o "${UID}" -g "${GID}" /workspaces/empty

USER dev
WORKDIR /home/dev

RUN git clone --depth=1 https://github.com/ohmyzsh/ohmyzsh.git "$HOME/.oh-my-zsh" \
    && git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "$HOME/.oh-my-zsh/custom/themes/powerlevel10k" \
    && git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions "$HOME/.oh-my-zsh/custom/plugins/zsh-autosuggestions" \
    && git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting "$HOME/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting" \
    && git clone --depth=1 https://github.com/zsh-users/zsh-completions "$HOME/.oh-my-zsh/custom/plugins/zsh-completions" \
    && git clone --depth=1 https://github.com/fdellwing/zsh-bat.git "$HOME/.oh-my-zsh/custom/plugins/zsh-bat" \
    && cp "$HOME/.oh-my-zsh/custom/themes/powerlevel10k/config/p10k-rainbow.zsh" "$HOME/.p10k.zsh"

COPY --chown=1000:1001 .devcontainer/zshrc /home/dev/.zshrc

RUN curl -fsSL -o /tmp/resh.tar.gz "https://github.com/curusarn/resh/releases/download/v3.0.2/resh_3.0.2_linux_amd64.tar.gz" \
    && echo "f226584fbb1ba8bffc2259e68f3b869c2a9d85a8f3fa1d3d973dc6da4002e702  /tmp/resh.tar.gz" | sha256sum -c - \
    && mkdir -p /tmp/resh "$HOME/.local/share/resh" \
    && echo "dev-container" > "$HOME/.local/share/resh/device-name" \
    && tar -xzf /tmp/resh.tar.gz -C /tmp/resh \
    && cd /tmp/resh \
    && SHELL=/bin/zsh ./scripts/install.sh \
    && rm -rf /tmp/resh /tmp/resh.tar.gz

ARG CURSOR_AGENT_VERSION=2026.10.01-e373342
RUN install -d "$HOME/.local/share/cursor-agent/versions/${CURSOR_AGENT_VERSION}" "$HOME/.local/bin" \
    && curl -fSL "https://downloads.cursor.com/lab/${CURSOR_AGENT_VERSION}/linux/x64/agent-cli-package.tar.gz" \
      | tar --strip-components=1 -xzf - -C "$HOME/.local/share/cursor-agent/versions/${CURSOR_AGENT_VERSION}" \
    && ln -s "$HOME/.local/share/cursor-agent/versions/${CURSOR_AGENT_VERSION}/cursor-agent" "$HOME/.local/bin/cursor-agent" \
    && ln -s "$HOME/.local/bin/cursor-agent" "$HOME/.local/bin/agent"

RUN curl -fsSL https://opencode.ai/install | bash -s -- --version 1.18.30 --no-modify-path

USER root
RUN curl -fsSL -o /tmp/node.tar.xz "https://nodejs.org/dist/v22.23.3/node-v22.23.3-linux-x64.tar.xz" \
    && echo "df450af89261115ef9f9e3830c3eeb2cc9213b63c720b1af623cb5dcbe2e02de  /tmp/node.tar.xz" | sha256sum -c - \
    && tar -xJf /tmp/node.tar.xz -C /usr/local --strip-components=1 \
    && rm -f /tmp/node.tar.xz

USER dev
ENV PATH="/home/dev/.local/bin:/home/dev/.pi/agent/bin:/home/dev/.opencode/bin:/home/dev/.resh/bin:${PATH}"

RUN curl -fsSL https://pi.dev/install.sh | sh

RUN git clone --depth 1 --branch v0.40.8 https://github.com/nvm-sh/nvm.git "$HOME/.nvm" \
    && bash -c 'set -euo pipefail; export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"; nvm install 24.18.0; nvm alias default 24.18.0; npm install --global --prefix "$HOME/.local" rea-agents@6.3.0'

ENV PATH="/home/dev/.nvm/versions/node/v24.18.0/bin:${PATH}"

WORKDIR /workspaces/empty
CMD ["zsh"]
