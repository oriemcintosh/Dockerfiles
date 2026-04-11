# Dockerfile for AWS Development

FROM ubuntu:resolute-20260404@sha256:cc925e589b7543b910fea57a240468940003fbfc0515245a495dd0ad8fe7cef1

ARG TARGETARCH

# Change Working Directory to Home
WORKDIR /home

# Run Updates
RUN apt update

# Install vim
RUN apt install vim -y

# Install git
RUN apt install git -y

# Install unzip
RUN apt install unzip -y

# Install jq
RUN apt install jq -y

# Install curl
RUN apt install curl -y

# Install wget
RUN apt install wget -y

# Install zsh
RUN apt install zsh -y

# Install Python3, pip, and Python3-venv
RUN apt install python3 -y
RUN apt install python3-pip -y
RUN apt install python3-venv -y

# Install oh-my-zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Run upgrades
RUN apt upgrade -y

# Install Node.js LTS (system-wide) so pnpm and npm work without nvm
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt install -y nodejs

# Make Downloads Folder
RUN mkdir downloads

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
        && ./aws/install

# Install SAM CLI
RUN case "$TARGETARCH" in \
            amd64) sam_arch="x86_64" ;; \
    arm64) sam_arch="arm64" ;; \
            *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
        esac \
        && wget "https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-${sam_arch}.zip" \
        && unzip "aws-sam-cli-linux-${sam_arch}.zip" -d sam-installation \
        && ./sam-installation/install

# Install Azure CLI
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
RUN cp -r .oh-my-zsh /home/cloud-dev; chown cloud-dev:cloud-dev /home/cloud-dev/.zshrc

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

# Install uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# Start the container as cloud-dev
USER cloud-dev

# Change Working Directory to cloud-dev home
WORKDIR /home/cloud-dev
