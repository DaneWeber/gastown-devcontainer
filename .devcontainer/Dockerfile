FROM node:22-bookworm-slim AS node
FROM golang:1.26.10-bookworm

ARG GT_VERSION=1.2.1
# Keep the CLI aligned with the Beads library in Gas Town's go.mod.
ARG BD_VERSION=1.0.0
ARG DOLT_VERSION=2.7.1
ARG CLAUDE_VERSION=2.1.318

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl git libicu-dev openssh-client pkg-config \
    ripgrep sqlite3 sudo tmux \
    && rm -rf /var/lib/apt/lists/*

COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx \
    && npm install -g "@anthropic-ai/claude-code@${CLAUDE_VERSION}" \
    && npm cache clean --force

RUN set -eux; \
    arch="$(dpkg --print-architecture)"; \
    case "$arch" in amd64|arm64) ;; *) echo "Unsupported architecture: $arch"; exit 1 ;; esac; \
    curl -fsSL "https://github.com/dolthub/dolt/releases/download/v${DOLT_VERSION}/dolt-linux-${arch}.tar.gz" -o /tmp/dolt.tar.gz; \
    tar -xzf /tmp/dolt.tar.gz -C /tmp; \
    install "/tmp/dolt-linux-${arch}/bin/dolt" /usr/local/bin/dolt; \
    rm -rf /tmp/dolt*

RUN GOBIN=/usr/local/bin go install "github.com/steveyegge/gastown/cmd/gt@v${GT_VERSION}" \
    && GOBIN=/usr/local/bin go install "github.com/steveyegge/beads/cmd/bd@v${BD_VERSION}" \
    && go clean -cache -modcache

RUN useradd --create-home --shell /bin/bash --uid 1000 vscode \
    && echo 'vscode ALL=(root) NOPASSWD:ALL' > /etc/sudoers.d/vscode \
    && chmod 0440 /etc/sudoers.d/vscode \
    && mkdir -p /home/vscode/gt \
    && chown -R vscode:vscode /home/vscode /go

ENV PATH="/home/vscode/.local/bin:/home/vscode/go/bin:/usr/local/go/bin:${PATH}" \
    GOPATH="/home/vscode/go" \
    TERM=xterm-256color

USER vscode
WORKDIR /home/vscode
CMD ["sleep", "infinity"]
