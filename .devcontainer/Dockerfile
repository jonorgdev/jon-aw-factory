# Use Ubuntu as the base image
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Install dependencies
RUN apt-get update && \
    apt-get install -y curl git gh && \
    rm -rf /var/lib/apt/lists/*

# Install gh-aw CLI extension
RUN curl -sL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash

# Set up workspace
WORKDIR /workspace

# Install VS Code extensions (for devcontainer)
# These are installed automatically by devcontainer.json, but you can add more here if needed

# Default shell
CMD ["/bin/bash"]
