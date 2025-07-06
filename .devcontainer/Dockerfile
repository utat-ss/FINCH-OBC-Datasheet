FROM debian:buster

# practical minimum
RUN apt-get update && apt-get install -y make build-essential git zsh vim curl wget procps
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.2/zsh-in-docker.sh)" -- \
    -t robbyrussell -p git -p ssh-agent

# install tex live 2025
ARG TL2025_ISO_URL=https://ca.mirrors.cicku.me/ctan/systems/texlive/Images/texlive2025.iso
ARG TL2025_ISO_SHA512="1fd9a2234d086f50c832ab3ac8b83477bbc39b1138a3c4a2351244c1fd4b8bea6d1ac81d4a5b0cba95f2e82c00f0d9df5b33189eb222e4bae5dae1523ef0da0e  texlive2025.iso"

RUN apt-get update && apt-get install -y perl-tk cpanminus xorriso
# RUN wget ${TL2022_ISO_URL} --quiet --show-progress --progress=bar:force:noscroll
ADD ${TL2025_ISO_URL} /texlive2025.iso
RUN echo ${TL2025_ISO_SHA512} | sha512sum -c
RUN osirrox -indev texlive2025.iso -extract / /tmp/texlive2025
RUN cd /tmp/texlive2025 && bash -c "echo 'I' | ./install-tl"

ENV PATH="$PATH:/usr/local/texlive/2025/bin/x86_64-linux"

RUN rm /texlive2025.iso
RUN rm -rf /tmp/texlive2025

# fix latexindent.pl
RUN cpanm YAML::Tiny Unicode::GCString File::HomeDir
