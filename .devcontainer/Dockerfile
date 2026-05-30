# Build stage for TeX Live
FROM public.ecr.aws/lts/ubuntu:22.04_stable AS texlive-builder

ARG DEBIAN_FRONTEND=noninteractive

# Update and install build dependencies
RUN apt update && apt upgrade -y && \
    apt install -y \
    perl-tk cpanminus xorriso wget

# Install tex live 2026
ARG TL2026_ISO_NAME=texlive2026-20260301.iso
ARG TL2026_ISO_URL=https://ca.mirrors.cicku.me/ctan/systems/texlive/Images/${TL2026_ISO_NAME}
ARG TL2026_ISO_SHA512="4a9071bb567c3bdd6443378dedc8e485aea4a2f1203ec8ed7c17f6787093b9c37636a037032c0be63352e3d0bf98cf5616dab19fdcd7cb83f766b3e085b620ff  ${TL2026_ISO_NAME}"

RUN cd /tmp && wget ${TL2026_ISO_URL} --quiet --show-progress --progress=bar:force:noscroll
RUN cd /tmp && echo ${TL2026_ISO_SHA512} | sha512sum -c
RUN cd /tmp && osirrox -indev ${TL2026_ISO_NAME} -extract / /tmp/texlive2026
RUN cd /tmp/texlive2026 && bash -c "echo 'I' | ./install-tl"

# main stage
FROM public.ecr.aws/lts/ubuntu:22.04_stable

ARG DEBIAN_FRONTEND=noninteractive
ARG KEYRING_PATH=/usr/share/keyrings
ARG APT_SOURCES_PATH=/etc/apt/sources.list.d

# update and upgrade
RUN apt update && apt upgrade -y

# install essentials
RUN apt update && \
    apt install -y \
    man make build-essential cmake ninja-build git zsh vim curl wget procps gnupg gnupg2 ca-certificates zip \
    software-properties-common perl-tk cpanminus

# unminimize the system
RUN bash -c "yes | unminimize"

# create dev sudo user
RUN useradd --create-home dev && \
    usermod --append --groups sudo dev && \
    apt update && apt install -y sudo && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# install dumb init system
USER root
RUN apt update && apt install -y dumb-init

# Copy TeX Live from builder stage
COPY --from=texlive-builder /usr/local/texlive/2026 /usr/local/texlive/2026

# setup oh-my-zsh
USER dev
ARG DOCKER_OHMYZSH_SCRIPT_NAME=zsh-in-docker.sh
ARG DOCKER_OHMYZSH_SCRIPT_URL=https://github.com/deluan/zsh-in-docker/releases/download/v1.1.3/${DOCKER_OHMYZSH_SCRIPT_NAME}
ARG DOCKER_OHMYZSH_SCRIPT_HASH="ffa8175332ef01b500ace59d03ce7e2f3a7453651e9a37060974bb6536f0706b  ${DOCKER_OHMYZSH_SCRIPT_NAME}"
RUN cd /tmp && \
    wget ${DOCKER_OHMYZSH_SCRIPT_URL} && \
    echo ${DOCKER_OHMYZSH_SCRIPT_HASH} | sha256sum -c && \
    chmod +x ./${DOCKER_OHMYZSH_SCRIPT_NAME} && \
    ./${DOCKER_OHMYZSH_SCRIPT_NAME} -t robbyrussell -p git -p ssh-agent && \
    rm ./${DOCKER_OHMYZSH_SCRIPT_NAME}

ENV PATH="$PATH:/usr/local/texlive/2026/bin/x86_64-linux"

# fix latexindent.pl
RUN sudo cpanm YAML::Tiny Unicode::GCString File::HomeDir

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
