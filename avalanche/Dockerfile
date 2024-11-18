ARG AVALANCHE_HOME="/home/.avalanchego"

ARG AVALANCHE_REPO="https://github.com/ava-labs/avalanchego.git"
ARG AVALANCHE_RELEASE="v1.11.12"

ARG AVALANCHE_SUBNETS_REPO="https://github.com/ava-labs/subnet-evm"
ARG AVALANCHE_SUBNETS_RELEASE="v0.6.10"

ARG AVALANCHE_CHAIN_ASSETS_REPO="https://github.com/ava-labs/public-chain-assets"

ARG DFK_ETH_CHAIN_ID="53935"
ARG DFK_VM_ID="mDV3QWRXfwgKUWb9sggkv4vQxAQR4y2CyKrt5pLZ5SzQ7EHBv"
ARG DFK_BLOCKCHAIN_ID="q2aTwKuyzgs8pynF7UXBZCU7DejbZbZ6EUyHr3JQzYgwNPUPi"


FROM golang:1.22.8-bullseye AS builder

ARG AVALANCHE_REPO
ARG AVALANCHE_RELEASE

ARG AVALANCHE_SUBNETS_REPO
ARG AVALANCHE_SUBNETS_RELEASE

ARG AVALANCHE_CHAIN_ASSETS_REPO

ARG DFK_VM_ID

RUN apt-get update && \
    apt-get install -y --no-install-recommends git bash make gcc musl-dev ca-certificates linux-headers-amd64

# Build Avalanche
WORKDIR /avalanchego

RUN git clone --depth 1 -b ${AVALANCHE_RELEASE} ${AVALANCHE_REPO} .

RUN scripts/build.sh

# Build subnets
WORKDIR /subnet-evm

RUN git clone --depth 1 -b ${AVALANCHE_SUBNETS_RELEASE} ${AVALANCHE_SUBNETS_REPO} .

RUN ./scripts/build.sh /avalanchego/build/plugins/${DFK_VM_ID}

# Get chain assets
WORKDIR /public-chain-assets

RUN git clone --depth 1 ${AVALANCHE_CHAIN_ASSETS_REPO} .

FROM debian:bullseye-slim as execution

ARG DFK_ETH_CHAIN_ID
ARG DFK_BLOCKCHAIN_ID
ARG AVALANCHE_HOME

ENV DEBIAN_FRONTEND=noninteractive \
    USERNAME=appuser \
    APP_PATH=/data

RUN apt update && \
    apt -y dist-upgrade && \
    apt install -y --no-install-recommends \
        tzdata \
        ca-certificates && \
    rm -rf /var/cache/apt/*

RUN groupadd -g 1001 ${USERNAME} \
    && useradd -m -d ${APP_PATH} -u 1001 -g 1001 ${USERNAME}

# Copy avalanchego binary to /usr/local/bin
COPY --from=builder /avalanchego/build/avalanchego /usr/local/bin/

# Copy plugins to /usr/local/lib/avalanchego/plugins
COPY --from=builder /avalanchego/build/plugins /usr/local/lib/avalanchego/plugins

# Copy upgrade.json
COPY --from=builder /public-chain-assets/chains/${DFK_ETH_CHAIN_ID}/upgrade.json ${AVALANCHE_HOME}/configs/chains/${DFK_BLOCKCHAIN_ID}/upgrade.json

COPY ./config.json ${AVALANCHE_HOME}/configs/config.json
COPY ./chains/ ${AVALANCHE_HOME}/configs/chains/

VOLUME ${APP_PATH}

WORKDIR ${APP_PATH}
USER ${USERNAME}

ENTRYPOINT ["avalanchego"]
