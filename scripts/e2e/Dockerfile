ARG RUST_VERSION=1.85
ARG FEATURES

FROM docker.io/rust:${RUST_VERSION}-slim-bookworm

RUN apt-get update && apt-get -y --no-install-recommends install \
    build-essential \
    cmake \
    curl \
    git \
    clang \
    libclang-dev \
    libsasl2-dev \
    libstdc++-11-dev \
    libssl-dev \
    libxxhash-dev \
    unzip \
    zlib1g-dev \
    zlib1g

RUN git clone https://github.com/rui314/mold.git \
    && mkdir mold/build \
    && cd mold/build \
    && git checkout v2.0.0 \
    && ../install-build-deps.sh \
    && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=c++ .. \
    && cmake --build . -j $(nproc) \
    && cmake --install .

RUN cargo install cargo-nextest --version 0.9.95 --locked

COPY scripts/environment/install-protoc.sh /
COPY tests/data/ca/certs /certs
RUN bash /install-protoc.sh

WORKDIR /vector
COPY . .
ARG FEATURES

RUN --mount=type=cache,target=/vector/target \
    --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/usr/local/cargo/git \
    /usr/local/bin/mold -run cargo build --tests --lib --bin vector \
    --no-default-features --features $FEATURES && \
    cp target/debug/vector /usr/bin/vector
