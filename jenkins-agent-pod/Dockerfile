FROM jenkins/inbound-agent:latest

USER root

# Install tools
RUN apt-get update && \
    apt-get install -y curl git ca-certificates && \
    rm -rf /var/lib/apt/lists/*


# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/v1.33.0/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl


# Install Kaniko executor
RUN mkdir -p /kaniko && \
    curl -L https://github.com/GoogleContainerTools/kaniko/releases/download/v1.23.2/executor-linux-amd64 \
    -o /kaniko/executor && \
    chmod +x /kaniko/executor


# Return to Jenkins user
USER jenkins