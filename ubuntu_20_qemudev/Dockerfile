FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive

ARG UID=1000

RUN sed -i.org -e 's|ports.ubuntu.com|jp.archive.ubuntu.com|g' /etc/apt/sources.list \
    && apt-get update && apt-get upgrade -y && apt-get install -y \
       sudo \
       iputils-ping \
       iproute2 \
       x11-apps \
       curl \
       lsb-core \
       git \
       vim \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m -u ${UID} user
RUN mkdir -p /etc/sudoers.d \
    && touch /etc/sudoers.d/user
RUN echo "%user	ALL=(ALL)	NOPASSWD:	ALL" >> /etc/sudoers.d/user \
    && chmod 0440 /etc/sudoers.d/user

RUN apt-get update && apt-get install -y \
    build-essential \
    g++-arm-linux-gnueabihf \
    qemu-user-binfmt

RUN chsh -s /bin/bash user
USER user
WORKDIR /home/user

ENV RUST_VERSION stable
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain ${RUST_VERSION}
ENV PATH $PATH:$HOME/.cargo/bin
