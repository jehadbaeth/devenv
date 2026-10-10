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

USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
      findutils \
      iptables \
      zip \
    && rm -rf /var/lib/apt/lists/*

COPY .devcontainer/lock-net /usr/local/bin/lock-net
RUN chmod 755 /usr/local/bin/lock-net

USER dev
RUN curl -fsSL -o /tmp/sdkman-install.sh https://get.sdkman.io \
    && echo "151a6fa9494053deda70ae54f93c2edf43716ddf0da2a1033c1d65a19a24b78e  /tmp/sdkman-install.sh" | sha256sum -c - \
    && bash /tmp/sdkman-install.sh \
    && rm -f /tmp/sdkman-install.sh \
    && sed -i \
      -e 's/sdkman_auto_answer=false/sdkman_auto_answer=true/' \
      -e 's/sdkman_selfupdate_feature=true/sdkman_selfupdate_feature=false/' \
      -e 's/sdkman_healthcheck_enable=true/sdkman_healthcheck_enable=false/' \
      -e 's/sdkman_curl_connect_timeout=7/sdkman_curl_connect_timeout=30/' \
      -e 's/sdkman_curl_max_time=10/sdkman_curl_max_time=300/' \
      "$HOME/.sdkman/etc/config" \
    && bash -c 'set -eo pipefail; source "$HOME/.sdkman/bin/sdkman-init.sh"; sdk install java "21.0.12+1.1-tem"'

# The global -h alias in zshrc rewrites SDKMAN's [[ -h ]] tests.
USER root
COPY .devcontainer/enable-sdkman /usr/local/bin/enable-sdkman
RUN chmod 755 /usr/local/bin/enable-sdkman
USER dev
RUN enable-sdkman

USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
      autoconf \
      automake \
      cmake \
      file \
      fontconfig \
      fonts-dejavu-core \
      gdb \
      libffi-dev \
      libssl-dev \
      libtool \
      libx11-6 \
      libxext6 \
      libxi6 \
      libxrender1 \
      libxtst6 \
      patch \
      procps \
      python-is-python3 \
      python3 \
      python3-dev \
      python3-pip \
      python3-setuptools \
      python3-venv \
      python3-wheel \
      wget \
      zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fL -o /tmp/ghidra.zip "https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_12.1.4_build/ghidra_12.1.4_PUBLIC_20260921.zip" \
    && echo "ddac49f903da9d5bac833e5cc79395098b9c33cfd3279be5f31bd00387d2d4db  /tmp/ghidra.zip" | sha256sum -c - \
    && unzip -q /tmp/ghidra.zip -d /opt \
    && mv /opt/ghidra_12.1.4_PUBLIC /opt/ghidra \
    && rm -f /tmp/ghidra.zip \
    && chmod -R a+rX /opt/ghidra

COPY .devcontainer/ghidra /usr/local/bin/ghidra
COPY .devcontainer/analyzeHeadless /usr/local/bin/analyzeHeadless
RUN chmod 755 /usr/local/bin/ghidra /usr/local/bin/analyzeHeadless

USER dev
RUN npm config set prefix "$HOME/.local" \
    && mkdir -p "$HOME/.config/pip" \
    && printf '%s\n' '[global]' 'break-system-packages = true' 'user = true' > "$HOME/.config/pip/pip.conf" \
    && npm install --global --prefix "$HOME/.local" @anthropic-ai/claude-code@2.1.296 \
    && pi install npm:pi-agent-goal@2026.7.18 \
    && pi install npm:pi-cursor-sdk@0.5.3 \
    && pi install npm:pi-editor-cursor@0.1.2 \
    && pi install npm:@thoth-agents/pi-questions-user@0.3.0 \
    && pi install npm:pi-mcp-adapter@5.2.0 \
    && pi install npm:pi-subagents@0.77.0 \
    && pi install npm:pi-web-access@0.38.0 \
    && pi install npm:@juicesharp/rpiv-ask-user-question@2.12.0 \
    && pi install npm:pi-goal-x@0.32.3 \
    && pi install npm:bigpowers@2.88.9 \
    && pi install npm:@plannotator/pi-extension@0.28.11 \
    && pi install npm:pi-lens@4.4.2

# These pairs register the same tools. Keep one of each, and leave MCP to pi-mcp-adapter.
RUN pi remove npm:pi-goal-x@0.32.3 \
    && pi remove npm:@juicesharp/rpiv-ask-user-question@2.12.0 \
    && python3 -c 'import json; from pathlib import Path; p=Path.home()/".pi/agent/settings.json"; data=json.loads(p.read_text()); ext=list(data.get("extensions") or []); ext.append("-builtin:mcp") if "-builtin:mcp" not in ext else None; data["extensions"]=ext; p.write_text(json.dumps(data, indent=2)+"\n")'

# nvm rejects a prefix in ~/.npmrc. Global installs use nvm's own Node, which the user can write.
RUN rm -f "$HOME/.npmrc"

ENV PATH="/home/dev/.sdkman/candidates/java/current/bin:${PATH}"

USER root
RUN curl -fsSL https://apt.llvm.org/llvm-snapshot.gpg.key -o /etc/apt/trusted.gpg.d/apt.llvm.org.asc \
    && printf '%s\n' 'deb [signed-by=/etc/apt/trusted.gpg.d/apt.llvm.org.asc] https://apt.llvm.org/bookworm/ llvm-toolchain-bookworm-18 main' > /etc/apt/sources.list.d/llvm.list \
    && apt-get update && apt-get install -y --no-install-recommends \
      clang-18 \
      libc++-18-dev \
      libc++abi-18-dev \
      libcapstone-dev \
      libicu-dev \
      ninja-build \
      python3-capstone \
      python3-pyelftools \
      python3-requests \
    && rm -rf /var/lib/apt/lists/* \
    && ln -sf /usr/bin/clang-18 /usr/local/bin/clang

COPY .devcontainer/clang++-libcxx /usr/local/bin/clang++-libcxx
COPY .devcontainer/blutter /usr/local/bin/blutter
RUN chmod 755 /usr/local/bin/clang++-libcxx /usr/local/bin/blutter \
    && ln -sf /usr/local/bin/clang++-libcxx /usr/local/bin/clang++ \
    && printf '%s\n' '#include <format>' 'int main(){return 0;}' | /usr/local/bin/clang++-libcxx -std=c++20 -x c++ - -o /tmp/format-test \
    && rm -f /tmp/format-test \
    && pkg-config --exists icu-uc capstone

ENV CC=/usr/bin/clang-18 \
    CXX=/usr/local/bin/clang++-libcxx

USER dev
RUN git clone --depth 1 https://github.com/worawit/blutter.git "$HOME/blutter" \
    && test "$(git -C "$HOME/blutter" rev-parse HEAD)" = "4a60ac648bf448c5a7596437243bcd0b9376fdf0" \
    && python3 -c 'import elftools, requests, capstone'

WORKDIR /workspaces/empty
CMD ["zsh"]
