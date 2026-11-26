FROM ubuntu:26.04 as baseimage2604
LABEL maintainer tpfuemp@github

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# build stage: toolchain, sources and intermediate files stay here
FROM baseimage2604 as builder

RUN apt-get update && \
    apt-get install -y --no-install-recommends build-essential git pkg-config patch && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Clone the source code.
ARG STRATUM_UPDATEPASS
# v1.0.9
ARG CURRENT_GITHUB_COMMIT=94fda624f0c941269d1f89ecc4bf6f3570064920
ARG RUST_VERSION=1.96.0
ARG DATASET_URL=https://github.com/SilasEigen/neb/releases/download/v1.0.9/neb-datasets.bin

# NEB is written in Rust (edition 2024), the Ubuntu rustc is too old
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --profile minimal --default-toolchain $RUST_VERSION
ENV PATH="/root/.cargo/bin:${PATH}"

RUN git clone https://github.com/SilasEigen/neb
WORKDIR /neb

# use the specific commit known to work
RUN git checkout $CURRENT_GITHUB_COMMIT

# apply patch: --rpc-bind / --work-bind (JSON-RPC and work service bind address, default 127.0.0.1)
COPY docker.patch /tmp/
RUN patch -p1 < /tmp/docker.patch

RUN mkdir -p /out && cd node && cargo build --release --locked -p neb-node -p neb-wallet && \
    strip target/release/neb-node target/release/neb-wallet && \
    cp target/release/neb-node target/release/neb-wallet /out/

# the node needs the dataset file (pinned by hash, the node refuses another file)
RUN curl --fail --location --silent --show-error --retry 3 -o /out/neb-datasets.bin $DATASET_URL && \
    /out/neb-node --dataset-hash /out/neb-datasets.bin

# blocknotify (kept for consistency, neb-node has no -blocknotify option)
COPY blocknotify.cpp /tmp/
RUN gcc -DSTRATUM_UPDATEPASS=\"$STRATUM_UPDATEPASS\" -o /out/blocknotify /tmp/blocknotify.cpp

# runtime stage: binaries and dataset only
FROM baseimage2604 as coinwallet

COPY --from=builder /out/neb-node /out/neb-wallet /out/blocknotify /usr/bin/
COPY --from=builder /out/neb-datasets.bin /usr/share/neb/

ARG walletuserid
ARG walletgroupid
RUN groupadd -g $walletgroupid walletuser && useradd -r -u $walletuserid -g walletuser walletuser
USER walletuser 
