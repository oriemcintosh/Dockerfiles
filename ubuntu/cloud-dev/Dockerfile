# Dockerfile for AWS Development

FROM ubuntu:resolute-20260404@sha256:cc925e589b7543b910fea57a240468940003fbfc0515245a495dd0ad8fe7cef1

ARG TARGETARCH

# Change Working Directory to Home
WORKDIR /home

# Install base packages
RUN apt-get update \
    && apt-get install -y \
        vim \
        git \
        unzip \
        jq \
        curl \
        wget \
        zsh \
        python3 \
        python3-pip \
        python3-venv \
    && rm -rf /var/lib/apt/lists/*

# Install oh-my-zsh
RUN RUNZSH=no CHSH=no KEEP_ZSHRC=yes sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Run upgrades
RUN apt-get update \
    && apt-get upgrade -y \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js LTS (system-wide) using official NodeSource GPG key and apt repository
RUN apt-get update \
    && apt-get install -y ca-certificates gnupg \
    && mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
       | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_22.x nodistro main" \
       > /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Make Downloads Folder
RUN mkdir -p /downloads

# Change Working Directory for Installs
WORKDIR /downloads

# Install AWS CLI
RUN case "$TARGETARCH" in \
            amd64) aws_arch="x86_64" ;; \
            arm64) aws_arch="aarch64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && curl "https://awscli.amazonaws.com/awscli-exe-linux-${aws_arch}.zip" -o "awscliv2.zip" \
        && unzip awscliv2.zip \
        && ./aws/install \
        && rm -rf awscliv2.zip aws

# Install SAM CLI
RUN case "$TARGETARCH" in \
            amd64) sam_arch="x86_64" ;; \
            arm64) sam_arch="arm64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && wget "https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-${sam_arch}.zip" \
        && unzip "aws-sam-cli-linux-${sam_arch}.zip" -d sam-installation \
        && ./sam-installation/install \
        && rm -rf "aws-sam-cli-linux-${sam_arch}.zip" sam-installation

# Install Azure CLI (use Microsoft's universal install script to avoid distro repo issues)
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Install Helm
RUN case "$TARGETARCH" in \
            amd64) helm_arch="amd64" ;; \
            arm64) helm_arch="arm64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && curl -fsSL "https://get.helm.sh/helm-v3.16.4-linux-${helm_arch}.tar.gz" -o /tmp/helm.tar.gz \
        && tar -xzf /tmp/helm.tar.gz -C /tmp \
        && mv /tmp/linux-${helm_arch}/helm /usr/local/bin/helm \
        && chmod +x /usr/local/bin/helm \
        && rm -rf /tmp/helm.tar.gz /tmp/linux-${helm_arch}

# Install kubectl
RUN case "$TARGETARCH" in \
            amd64) kubectl_arch="amd64" ;; \
            arm64) kubectl_arch="arm64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && curl -fsSL "https://dl.k8s.io/release/v1.31.2/bin/linux/${kubectl_arch}/kubectl" -o /usr/local/bin/kubectl \
        && chmod +x /usr/local/bin/kubectl

# Install Terraform and Terragrunt (latest releases)
RUN case "$TARGETARCH" in \
            amd64) tf_arch="amd64" ;; \
            arm64) tf_arch="arm64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && tf_version="$(curl -fsSL https://checkpoint-api.hashicorp.com/v1/check/terraform | jq -r .current_version)" \
        && echo "Installing terraform ${tf_version} for ${tf_arch}" \
        && curl -fsSL "https://releases.hashicorp.com/terraform/${tf_version}/terraform_${tf_version}_linux_${tf_arch}.zip" -o /tmp/terraform.zip \
        && unzip /tmp/terraform.zip -d /usr/local/bin \
        && chmod +x /usr/local/bin/terraform \
        && rm -f /tmp/terraform.zip \
        && tg_version="$(curl -fsSL https://api.github.com/repos/gruntwork-io/terragrunt/releases/latest | jq -r .tag_name | sed 's/^v//')" \
        && echo "Installing terragrunt ${tg_version} for ${tf_arch}" \
        && curl -fsSL -L "https://github.com/gruntwork-io/terragrunt/releases/download/v${tg_version}/terragrunt_linux_${tf_arch}" -o /usr/local/bin/terragrunt \
        && chmod +x /usr/local/bin/terragrunt

# Install latest Go at build time from go.dev
RUN case "$TARGETARCH" in \
            amd64) go_arch="amd64" ;; \
            arm64) go_arch="arm64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && go_version="$(curl -fsSL https://go.dev/VERSION?m=text | head -n 1 | tr -d '\r\n')" \
        && curl -fsSL "https://go.dev/dl/${go_version}.linux-${go_arch}.tar.gz" -o /tmp/go.tar.gz \
        && rm -rf /usr/local/go \
        && tar -C /usr/local -xzf /tmp/go.tar.gz \
        && rm -f /tmp/go.tar.gz

    # Make Go available system-wide
    ENV PATH="/usr/local/go/bin:${PATH}"

# Install Power Level 10k
RUN git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# Change Working Directory for zsh Changes
WORKDIR /root

# Configure AWS CLI Command Completion
RUN echo '\n#### AWS CLI Command Completion Helper ####\nexport PATH=/usr/local/bin:$PATH' >> .zshrc
RUN echo 'autoload bashcompinit && bashcompinit\nautoload -Uz compinit && compinit' >> .zshrc
RUN echo "complete -C '/usr/local/bin/aws_completer' aws" >> .zshrc

# Update .zshrc file for PowerLevel10k
RUN sed -i -e 's|ZSH_THEME="robbyrussell"|ZSH_THEME="powerlevel10k/powerlevel10k"|g' .zshrc

# Add zsh to .bashrc for startup / setup
RUN echo '\n#### Run zsh ####\nzsh' >> .bashrc

# Create cloud-dev user and copy SHELL files
RUN useradd -ms /bin/zsh cloud-dev
RUN mkdir /home/cloud-dev/downloads; chown cloud-dev:cloud-dev /home/cloud-dev/downloads
RUN cp -r .zshrc /home/cloud-dev; chown cloud-dev:cloud-dev /home/cloud-dev/.zshrc
RUN cp -r .oh-my-zsh /home/cloud-dev; chown -R cloud-dev:cloud-dev /home/cloud-dev/.oh-my-zsh

# Install uv from pinned release with hardcoded checksum verification (system-wide)
RUN UV_VERSION="0.6.14" \
    && case "$TARGETARCH" in \
        amd64) uv_arch="x86_64" ; uv_sha256="0aaf451c391d3913823bfb8ed354b446dcfd0553a32ed8266611e4181c61fd51" ;; \
        arm64) uv_arch="aarch64" ; uv_sha256="ea25597354af186bdd55aee0de431e16d45d82951a4f41f065a8e4dc27885265" ;; \
        *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
    esac \
    && curl -fsSL "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${uv_arch}-unknown-linux-gnu.tar.gz" \
       -o /tmp/uv.tar.gz \
    && echo "${uv_sha256}  /tmp/uv.tar.gz" | sha256sum -c - \
    && tar -xzf /tmp/uv.tar.gz -C /tmp \
    && mv "/tmp/uv-${uv_arch}-unknown-linux-gnu/uv" /usr/local/bin/uv \
    && mv "/tmp/uv-${uv_arch}-unknown-linux-gnu/uvx" /usr/local/bin/uvx \
    && chmod +x /usr/local/bin/uv /usr/local/bin/uvx \
    && rm -rf /tmp/uv.tar.gz "/tmp/uv-${uv_arch}-unknown-linux-gnu"

# Run the following as cloud-dev user
USER cloud-dev

# Make user-local binaries available in the shell
ENV PATH="/home/cloud-dev/.local/bin:/home/cloud-dev/.npm-global/bin:/home/cloud-dev/.local/share/pnpm:${PATH}"
ENV NPM_CONFIG_PREFIX=/home/cloud-dev/.npm-global
ENV PNPM_HOME=/home/cloud-dev/.local/share/pnpm

# Change Working Directory to cloud-dev downloads
WORKDIR /home/cloud-dev/downloads

# Create a user-owned npm prefix for global installs
RUN mkdir -p /home/cloud-dev/.npm-global

# Install pnpm
ENV SHELL=/usr/bin/zsh
RUN npm install -g pnpm
RUN pnpm env use --global lts

# (Removed nvm installation; using system Node and pnpm to select LTS)

# Install AWS CDK
SHELL ["/usr/bin/zsh", "-c"]
RUN . /home/cloud-dev/.zshrc \
    && npm install -g aws-cdk

# Start the container as cloud-dev
USER cloud-dev

# Change Working Directory to cloud-dev home
WORKDIR /home/cloud-dev
