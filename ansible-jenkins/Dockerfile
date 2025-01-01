
FROM jenkins/jenkins:lts-jdk17
USER root
# Install Docker cli
RUN curl -fsSL https://get.docker.com | sh
# Create docker group and add Jenkins user
RUN groupadd docker && usermod -aG docker jenkins
# Install curl (required for downloading kubectl)
RUN apt-get update && apt-get install -y curl
# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/










