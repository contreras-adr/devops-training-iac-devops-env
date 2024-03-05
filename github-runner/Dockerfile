FROM ubuntu:22.04

ARG RUNNER_VERSION="2.313.0"

# Set environment variables for configuration and defaults
ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Europe/Madrid

# # Install required packages
# RUN apt-get update && \
#     apt-get install -y \
#     apt-transport-https \
#     ca-certificates \
#     gnupg \
#     lsb-release \
#     curl jq build-essential libssl-dev libffi-dev python3 python3-venv python3-dev python3-pip

# # Add Docker's official GPG key
# RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 
# # Add Docker repository
# RUN echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
 
# # Install Docker CE CLI
# RUN apt-get update && \
#     apt-get install -y docker-ce-cli


RUN apt update -y && apt upgrade -y && useradd -m docker
RUN cd /home/docker && mkdir actions-runner && cd actions-runner \
    && curl -O -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz \
    && tar xzf ./actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz


RUN chown -R docker ~docker && /home/docker/actions-runner/bin/installdependencies.sh

COPY start.sh start.sh

# make the script executable
RUN chmod +x start.sh

# since the config and run script for actions are not allowed to be run by root,
# set the user to "docker" so all subsequent commands are run as the docker user
USER docker

ENTRYPOINT ["./start.sh"]