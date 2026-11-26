FROM ubuntu:22.04

# Set environment variables
ENV DEBIAN_FRONTEND="noninteractive" TZ="Asia/Kolkata"
ENV PYENV_ROOT="/home/shakti/.pyenv"
ENV PATH="$PYENV_ROOT/bin:$PYENV_ROOT/shims:$PATH"

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    make build-essential libssl-dev zlib1g-dev libbz2-dev \
    libreadline-dev libsqlite3-dev wget curl llvm \
    libncurses5-dev libncursesw5-dev xz-utils tk-dev \
    libffi-dev liblzma-dev git tcl-dev pkg-config \
    autoconf gperf flex bison \
    ghc libghc-regex-compat-dev libghc-syb-dev \
    libghc-old-time-dev libghc-split-dev openssh-client openssh-server\
    ca-certificates sudo passwd && \
    rm -rf /var/lib/apt/lists/*

# Create user and set password
RUN useradd -ms /bin/bash shakti && \
    echo 'shakti:shakti123' | chpasswd || true && \
    usermod -aG sudo shakti && \
    echo "shakti ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/shakti && \
    chmod 0440 /etc/sudoers.d/shakti

# Switch to user shakti
USER shakti
WORKDIR /home/shakti

# Install pyenv
RUN curl https://pyenv.run | bash

# Initialize pyenv
RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc && \
    echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc && \
    echo 'eval "$(pyenv init --path)"' >> ~/.bashrc && \
    echo 'eval "$(pyenv init -)"' >> ~/.bashrc && \
    echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc

# Install Python and create virtualenv
RUN bash -l -c "\
    CONFIGURE_OPTS=--enable-shared pyenv install 3.8.10 && \
    pyenv global 3.8.10 && \
    pyenv virtualenv 3.8.10 py38"

# Activate virtualenv and install pip tools
RUN bash -l -c "\
    export PYENV_ROOT=\"$HOME/.pyenv\" && \
    export PATH=\"$PYENV_ROOT/bin:$PYENV_ROOT/shims:$PATH\" && \
    eval \"\$(pyenv init --path)\" && \
    eval \"\$(pyenv init -)\" && \
    eval \"\$(pyenv virtualenv-init -)\" && \
    pyenv activate py38 && \
    python -m ensurepip && \
    pip install --upgrade pip setuptools wheel"

# Switch back to root for system-wide install of BSC
USER root
WORKDIR /opt

# Clone and install BSC
RUN git clone --recursive https://github.com/B-Lang-org/bsc && \
    cd bsc && git checkout 2021.12.1 && make install-src && \
    mv inst /opt/bsc-inst && cd /opt && rm -rf bsc

# Update PATH
ENV PATH="/opt/bsc-inst/bin:$PATH"

# Test BSC install
RUN bsc -help || { echo "BSC failed to install"; exit 1; }

# Final setup: return to user
USER shakti
WORKDIR /home/shakti

# Clone Shakti C-class cores
RUN git clone https://gitlab.com/shaktiproject/cores/c-class.git

# Default command
CMD ["/bin/bash"]
