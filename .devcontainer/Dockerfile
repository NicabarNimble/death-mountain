# YOLO Development Container
# Autonomous AI workspace with all detected toolchains

FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install Node.js for Claude Code CLI
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm@latest \
    && npm install -g pnpm@latest

# Install Claude Code CLI for autonomous AI work
RUN npm install -g @anthropic-ai/claude-code@latest \
    && mkdir -p /root/.claude-linux

# Configure Claude settings using official settings.json API
# YOLO mode with extended timeouts (1 hour for long builds)
RUN echo '{"permissions":{"defaultMode":"bypassPermissions","allow":[],"deny":[]},"env":{"BASH_DEFAULT_TIMEOUT_MS":3600000,"BASH_MAX_TIMEOUT_MS":3600000}}' > /root/.claude-linux/settings.json

# Set Claude config directory for bind-mounted credentials
RUN echo 'export CLAUDE_CONFIG_DIR=/root/.claude-linux' >> /etc/bash.bashrc

# Install 1Password CLI for secure credential management
RUN curl -sS https://downloads.1password.com/linux/keys/1password.asc | \
    gpg --dearmor --output /usr/share/keyrings/1password-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/1password-archive-keyring.gpg] https://downloads.1password.com/linux/debian/$(dpkg --print-architecture) stable main" | \
    tee /etc/apt/sources.list.d/1password.list && \
    mkdir -p /etc/debsig/policies/AC2D62742012EA22/ && \
    curl -sS https://downloads.1password.com/linux/debian/debsig/1password.pol | \
    tee /etc/debsig/policies/AC2D62742012EA22/1password.pol && \
    mkdir -p /usr/share/debsig/keyrings/AC2D62742012EA22 && \
    curl -sS https://downloads.1password.com/linux/keys/1password.asc | \
    gpg --dearmor --output /usr/share/debsig/keyrings/AC2D62742012EA22/debsig.gpg && \
    apt-get update && apt-get install -y 1password-cli


# Install asdf version manager for Dojo/Scarb version management
RUN mkdir -p /root/.local/bin && \
    wget -q https://github.com/asdf-vm/asdf/releases/download/v0.18.0/asdf-v0.18.0-linux-amd64.tar.gz && \
    tar -xzf asdf-v0.18.0-linux-amd64.tar.gz -C /root/.local/bin/ && \
    chmod +x /root/.local/bin/asdf && \
    rm asdf-v0.18.0-linux-amd64.tar.gz

# Configure asdf environment
ENV ASDF_DATA_DIR="/root/.asdf"
ENV PATH="/root/.local/bin:/root/.asdf/shims:$PATH"

# Install Scarb via asdf (multiple versions for compatibility)
RUN /root/.local/bin/asdf plugin add scarb && \
    /root/.local/bin/asdf install scarb 2.10.1 && \
    /root/.local/bin/asdf install scarb 2.12.2 && \
    /root/.local/bin/asdf reshim

# Install Dojo using dojoup (asdf-dojo plugin has bugs)
# Install latest as default, projects can specify version in .tool-versions for scarb
RUN curl -L https://install.dojoengine.org | bash && \
    . /root/.dojo/env && \
    dojoup install && \
    echo '. /root/.dojo/env' >> /etc/bash.bashrc

# Add asdf and Dojo to PATH
RUN echo 'export ASDF_DATA_DIR="/root/.asdf"' >> /etc/bash.bashrc && \
    echo 'export PATH="/root/.dojo/bin:/root/.local/bin:/root/.asdf/shims:$PATH"' >> /etc/bash.bashrc

# Set PATH for docker exec
ENV PATH="/root/.dojo/bin:/root/.local/bin:/root/.asdf/shims:$PATH"


# Create workspace directory
WORKDIR /workspace

# YOLO Mode indicator
ENV PATINA_YOLO=1

# Set up shell
CMD ["/bin/bash"]
