# SPDX-FileCopyrightText: (C) 2025 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

VERSION 0.8

# Include proxy settings
LOCALLY
ARG --global http_proxy=$(echo $http_proxy)
ARG --global https_proxy=$(echo $https_proxy)
ARG --global no_proxy=$(echo $no_proxy)
ARG --global HTTP_PROXY=$(echo $HTTP_PROXY)
ARG --global HTTPS_PROXY=$(echo $HTTPS_PROXY)
ARG --global NO_PROXY=$(echo $NO_PROXY)
ARG --global SOCKS_PROXY=$(echo $SOCKS_PROXY)

GIT_SETUP:
    FUNCTION

    RUN mkdir -p ~/.ssh/ && chmod 700 ~/.ssh && \
        echo "github.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm0SdG6UOoqKLsabgH5C9okWi0dh2l9GKJl" >> ~/.ssh/known_hosts
    IF [ -n "$SOCKS_PROXY" ]
        RUN echo "Host github.com
            ProxyCommand nc -x $SOCKS_PROXY %h %p" > ~/.ssh/config \
            && echo '#!/usr/bin/env sh
            exec nc -x $SOCKS_PROXY $1 $2' > /usr/bin/gitproxy \
            && chmod +x /usr/bin/gitproxy \
            && ( command -v git >/dev/null && git config --global core.gitproxy gitproxy || true )
    END
    IF [ -n "$HTTPS_PROXY" ]
        RUN if command -v git >/dev/null; then \
                git config --global http."https://github.com".proxy "$HTTPS_PROXY" \
                && git config --global http."https://lfs.github.com".proxy "$HTTPS_PROXY" \
                && git config --global http."https://github-cloud.s3.amazonaws.com".proxy "$HTTPS_PROXY" \
                && git config --global http."https://github-cloud.githubusercontent.com".proxy "$HTTPS_PROXY" \
            ; fi
    END
    RUN git config --global --add safe.directory '*'

edk-builder:
    FROM ubuntu:22.04
    RUN apt update && apt install -y \
        build-essential \
        uuid-dev \
        iasl \
        git \
        nasm \
        netcat \
        python-is-python3
    DO +GIT_SETUP
    WORKDIR /work

ovmf:
    FROM +edk-builder
    RUN git clone --depth 1 --recurse-submodules https://github.com/tianocore/edk2.git uefi/edk2
    WORKDIR uefi
    RUN make -C edk2/BaseTools -j $(nproc)
    RUN /usr/bin/bash -c 'export PACKAGES_PATH=$(pwd)/edk2:$(pwd); \
        pushd edk2 && source edksetup.sh && popd && \
        build -a X64 -b DEBUG -t GCC5 \
        -p edk2/OvmfPkg/OvmfPkgX64.dsc \
        -D NETWORK_HTTP_BOOT_ENABLE \
        -Y COMPILE_INFO \
        -y logs/report.txt'
    ENV TS=$(date +%s)
    SAVE ARTIFACT edk2/logs/report.txt AS LOCAL build/ovmf/${TS}/report.txt
    SAVE ARTIFACT edk2/Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd AS LOCAL build/ovmf/${TS}/OVMF.fd
    SAVE ARTIFACT edk2/Build/OvmfX64/DEBUG_GCC5/FV/OVMF_CODE.fd AS LOCAL build/ovmf/${TS}/OVMF_CODE.fd
    SAVE ARTIFACT edk2/Build/OvmfX64/DEBUG_GCC5/FV/OVMF_VARS.fd AS LOCAL build/ovmf/${TS}/OVMF_VARS.fd