---

# Cardano Node and CLI in Dockerized Environment

This project automates the setup of a Cardano Node and CLI within a containerized Ubuntu 22.04 environment using Docker. It installs all necessary dependencies and libraries to compile and run **Cardano Node** and **Cardano CLI**, offering a hassle-free development setup. The Dockerfile ensures reproducibility and consistency across different environments, making it easier for developers to work on Cardano-related projects.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Usage](#usage)
5. [Build Steps Explanation](#build-steps-explanation)
   - [Base Image](#1-base-image)
   - [System Dependencies](#2-system-dependencies)
   - [Haskell GHC Installation](#3-haskell-ghc-installation)
   - [Cabal Installation](#4-cabal-installation)
   - [Libsodium Setup](#5-libsodium-setup)
   - [secp256k1 Installation](#6-secp256k1-installation)
   - [blst Library Installation](#7-blst-library-installation)
   - [Cardano Node and CLI](#8-cardano-node-and-cli)
6. [Exposing Ports](#exposing-ports)
7. [Contact](#contact)
8. [License](#license)

---

## Project Overview

The purpose of this project is to streamline the installation and configuration of the **Cardano Node** and **Cardano CLI** in a Docker container, making it easier for developers to set up their development environment. The containerized environment ensures all dependencies are included and configured correctly, removing the need for manual setup.

With this project, you can quickly:
- Build and run **Cardano Node** in a containerized environment.
- Compile **Cardano CLI** for blockchain interaction.
- Use the provided tools and libraries to interact with the Cardano blockchain efficiently.

---

## Prerequisites

Before starting, ensure you have the following installed on your system:

- [Docker](https://docs.docker.com/get-docker/) (Version 19.03.0 or later)
- Adequate system resources:
  - **RAM**: Minimum 4 GB (8 GB recommended)
  - **Disk Space**: Minimum 30 GB (depending on the size of the blockchain you plan to interact with)
- Basic familiarity with Docker commands and Linux environment

---

## Installation

### Step 1: Clone the Repository

Clone this repository to your local machine to access the Dockerfile and associated files:

```bash
git clone https://github.com/rizwaanbashir2001/cardano-container-toolchain.git
cd cardano-container-toolchain
```

### Step 2: Build the Docker Image

Run the following command to build the Docker image:

```bash
docker build -t cardano-node .
```

This command will initiate the build process. The `Dockerfile` will download and install all the necessary dependencies, compile **Cardano Node** and **Cardano CLI**, and set up the environment.

---

## Usage

### Running the Docker Container

Once the Docker image is built, you can run the container using:

```bash
docker run -it --rm cardano-node
```

This command starts the container and drops you into an interactive shell, where you can begin using **Cardano Node** and **Cardano CLI** directly.

### Example Commands

Inside the container, you can start **Cardano Node** with the following command:

```bash
cardano-node run
```

You can also interact with **Cardano CLI** to query the blockchain:

```bash
cardano-cli query tip --testnet-magic 42
```

Make sure to replace the `--testnet-magic` parameter with the correct value for your network.

---

## Build Steps Explanation

Here’s a breakdown of the key steps and commands used to set up the environment:

### 1. Base Image

We start with the **Ubuntu 22.04** official base image:

```dockerfile
FROM ubuntu:22.04
```

This version of Ubuntu provides a stable and lightweight foundation for the Docker environment.

### 2. System Dependencies

We install a range of essential system dependencies required for building and running **Cardano Node**:

```dockerfile
RUN apt-get update -y && apt-get install -y \
    python3 \
    python3-pip \
    automake \
    build-essential \
    pkg-config \
    libffi-dev \
    libgmp-dev \
    libssl-dev \
    libtinfo-dev \
    systemd \
    libsystemd-dev \
    zlib1g-dev \
    npm \
    yarn \
    make \
    g++ \
    tmux \
    git \
    jq \
    wget \
    libncursesw5 \
    gnupg \
    libtool \
    autoconf \
    liblmdb-dev \
    libsodium-dev \
    libsodium23 \
    curl
```

These dependencies include compilers, package managers, system libraries, and build tools required to compile **Cardano Node**, **Cardano CLI**, and various cryptographic libraries.

### 3. Haskell GHC Installation

Next, we install **GHC 8.10.7**, which is a critical component for compiling **Cardano Node**:

```dockerfile
RUN wget https://downloads.haskell.org/ghc/8.10.7/ghc-8.10.7-x86_64-deb10-linux.tar.xz \
  && tar -xf ghc-8.10.7-x86_64-deb10-linux.tar.xz \
  && rm ghc-8.10.7-x86_64-deb10-linux.tar.xz \
  && cd ghc-* \
  && ./configure \
  && make install \
  && cd ..
```

We download the GHC tarball, extract it, and compile it from source using `make install`.

### 4. Cabal Installation

We install **Cabal 3.8.1.0**, the Haskell package manager, which is necessary to build **Cardano Node** and **Cardano CLI**:

```dockerfile
RUN wget https://downloads.haskell.org/~cabal/cabal-install-3.8.1.0/cabal-install-3.8.1.0-x86_64-linux-deb10.tar.xz
RUN tar -xf cabal-install-3.8.1.0-x86_64-linux-deb10.tar.xz
RUN rm cabal-install-3.8.1.0-x86_64-linux-deb10.tar.xz
RUN mv cabal /usr/bin/
RUN cabal update
```

We download the Cabal binary, move it to the system's `PATH`, and update the Cabal package list.

### 5. Libsodium Setup

The **Libsodium** cryptographic library is a requirement for **Cardano Node**. We install a specific custom version:

```dockerfile
RUN mkdir -p ~/src && cd ~/src && \
    git clone https://github.com/input-output-hk/libsodium && \
    cd libsodium && \
    git checkout dbb48cc && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make check && \
    make install && \
    echo "export LD_LIBRARY_PATH=\"/usr/local/lib:\$LD_LIBRARY_PATH\"" >> ~/.bashrc && \
    echo "export PKG_CONFIG_PATH=\"/usr/local/lib/pkgconfig:\$PKG_CONFIG_PATH\"" >> ~/.bashrc && \
    ldconfig
```

This step includes cloning the **Libsodium** repository, building it from source, and installing it system-wide. We also update environment variables to include the paths for the installed library.

### 6. secp256k1 Installation

We install **secp256k1**, a cryptographic library essential for elliptic curve operations:

```dockerfile
RUN cd ~/src && \
    git clone --depth 1 --branch v0.3.2 https://github.com/bitcoin-core/secp256k1 && \
    cd secp256k1 && \
    ./autogen.sh && \
    ./configure --enable-module-schnorrsig --enable-experimental && \
    make && \
    make check && \
    make install
```

This library is built with experimental features enabled.

### 7. blst Library Installation

The **blst** cryptographic library is also installed for **BLS12-381** signatures:

```dockerfile
RUN cd ~/src && \
    git clone --depth 1 --branch v0.3.11 https://github.com/supranational/blst && \
    cd blst && \
    ./build.sh && \
    ...
```

This step clones and builds the **blst** library from source, including custom configuration for the library paths.

### 8. Cardano Node and CLI

Finally, we clone the **Cardano Node** repository and build the **Cardano Node** and **Cardano CLI** binaries:

```dockerfile
RUN git clone --depth 1 --branch 8.7.3 https://github.com/input-output-hk/cardano-node.git && \
    cd cardano-node && \
    echo "with-compiler: ghc-8.10.7" >> cabal.project.local && \
    cabal update && \
    cabal build all && \
    cabal build cardano-cli
```

The binaries are built using Cabal and then copied to a directory in the user's `PATH`.

---

## Exposing Ports

The container exposes port `

3000`, which is the default port used by **Cardano Node**:

```dockerfile
EXPOSE 3000
```

---

## Contact

For any issues, suggestions, or queries, feel free to reach out:

- **Rizwaan Bashir**
  - [LinkedIn](https://www.linkedin.com/in/rizwaanbashir2001/)
  - Email: [rizwaanbashir2001@gmail.com](mailto:rizwaanbashir2001@gmail.com)

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

