FROM docker.io/library/golang:1

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt, install packages and tools
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git git-lfs iproute2 procps lsb-release jq \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Installs gopls and its dependencies. This provides full Go support for vscode.
RUN mkdir -p /tmp/gotools \
    && cd /tmp/gotools \
    # Not sure why, but these break in go 1.18+ if we build them all in one swoop
    && go install -v golang.org/x/tools/gopls@latest \
    && go install -v github.com/uudashr/gopkgs/v2/cmd/gopkgs@latest \
    && go install -v github.com/ramya-rao-a/go-outline@latest \
    && go install -v github.com/fatih/gomodifytags@latest \
    && go install -v github.com/haya14busa/goplay/cmd/goplay@latest \
    && go install -v github.com/josharian/impl@latest \
    && go install -v github.com/cweill/gotests/gotests@latest \
    && go install -v honnef.co/go/tools/cmd/staticcheck@latest \
    && go install -v golang.org/x/lint/golint@latest \
    && go install -v github.com/mgechev/revive@latest \
    && go install -v github.com/go-delve/delve/cmd/dlv@latest \
    #
    # The vscode-go team has been experimenting with dlv dap native interface. They expect the new binary to be named
    # dlv-dap, but it is actually identical to dlv. Yeah... I have no idea why as well... Lets just make a copy
    && ln -s /go/bin/dlv /go/bin/dlv-dap \
    #
    # Installs our extra tools (in addition to the ones above required for go vscode support)
    && go install -v github.com/vektra/mockery/v2@latest \
    && go install -v github.com/a-h/templ/cmd/templ@latest \
    #
    # Install golangci-lint (we dont use golint anymore, golangci will be the ongoing default linter)
    && curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin 2>&1 \
    #
    && curl -sSfL https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-$(go env GOOS)-$(go env GOARCH) -o /bin/tailwindcss \
    && chmod +x /bin/tailwindcss \
    # Nodejs stuff
    # && curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash - \
    # && apt-get install -y nodejs \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && go clean -cache \
    && rm -rf /var/lib/apt/lists/* /go/src /go/pkg /tmp/gotools \
    #
    # Add write permission for /go
    && chmod -R a+rwX /go

# Update this to "on" or "off" as appropriate
ENV GO111MODULE=auto

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog
