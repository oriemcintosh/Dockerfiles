# Node-base Dockerfile for AWS Development

FROM node:latest

# Run package manager updates
RUN apt update

# Install missing tools
RUN apt install vim jq python3-pip python3-venv -y

# Run upgrade on packages
RUN apt upgrade -y

# Create downloads directory
RUN mkdir /downloads

# Change working directory
WORKDIR /downloads

# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN ./aws/install

# Install SAM CLI
RUN wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
RUN unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
RUN ./sam-installation/install

# Insatll AWS CDK
RUN npm install -g aws-cdk