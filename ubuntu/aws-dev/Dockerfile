# Dockerfile for AWS Development

FROM ubuntu

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

# Make Downloads Folder
RUN mkdir downloads

# Change Working Directory for Installs
WORKDIR /downloads

# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN ./aws/install

# Install SAM CLI
RUN wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
RUN unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
RUN ./sam-installation/install

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

# Create aws-dev user and copy SHELL files
RUN useradd -ms /bin/zsh aws-dev
RUN mkdir /home/aws-dev/downloads; chown aws-dev:aws-dev /home/aws-dev/downloads
RUN cp -r .zshrc /home/aws-dev; chown aws-dev:aws-dev /home/aws-dev/.zshrc
RUN cp -r .oh-my-zsh /home/aws-dev; chown aws-dev:aws-dev /home/aws-dev/.zshrc

# Run the following as aws-dev user
USER aws-dev

# Change Working Directory to aws-dev downloads
WORKDIR /home/aws-dev/downloads


# Install NVM and Node LTS
ENV NVM_DIR=/home/aws-dev/.nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install --lts 

# Install AWS CDK
SHELL ["/usr/bin/zsh", "-c"]
RUN . /home/aws-dev/.zshrc \
    && npm install -g aws-cdk

# Install pnpm
ENV SHELL=/usr/bin/zsh
RUN curl -fsSL https://get.pnpm.io/install.sh | zsh -

# Change back to Root user
USER root

# Change Working Directory to aws-dev home
WORKDIR /home
