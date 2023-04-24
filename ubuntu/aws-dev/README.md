# AWS Development Dockerfile
This Dockerfile will include the tools an engineer / developer may use when working with AWS.

Some of the packages included are:
- AWS CLI
- SAM CLI
- AWS CDK
- NVM*
- pnpm*
- zsh / oh my zsh
  - Powerlevel 10k theme installed

This Dockerfile will also create a non-root user `aws-dev` for optional use. Packages with * are installed for non-root user.