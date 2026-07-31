FROM jenkins/inbound-agent:latest

USER root

# Install required tools
RUN apt-get update && \
    apt-get install -y \
    curl \
    git \
    ca-certificates \
    bash && \
    rm -rf /var/lib/apt/lists/*


# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/v1.33.0/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl


# Copy REAL Kaniko executor from official Kaniko image
COPY --from=gcr.io/kaniko-project/executor:v1.24.0 \
     /kaniko/executor \
     /kaniko/executor


# Prepare Kaniko for Jenkins user
RUN chmod +x /kaniko/executor && \
    mkdir -p /kaniko/.docker && \
    chown -R jenkins:jenkins /kaniko


USER jenkins