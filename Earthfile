VERSION --global-cache 0.8
PROJECT keinsell/neuronek-cli
IMPORT github.com/earthly/lib/rust:3.0.1 AS rust

CHECKOUT:
    FUNCTION
    WORKDIR /tmp/neuronek-cli
    COPY --keep-ts Cargo.toml ./
    COPY --keep-ts --dir src ./

builder:
    ARG IMAGE=registry.suse.com/bci/rust:1.82
    FROM $IMAGE
    DO rust+INIT --keep_fingerprints=true
    DO +CHECKOUT

lint:
    FROM +builder
    DO rust+CARGO --args="clippy --all-features --all-targets --offline -- -D warnings"

fmt:
    FROM +lint
    DO rust+CARGO --args="fmt --check"

test:
    FROM +builder
    DO rust+CARGO --args="test"

coverage:
    FROM +build
    RUN rustup component add llvm-tools
    DO rust+CARGO --args="llvm-cov --lcov --output-path lcov.info"
    DO rust+CARGO --args="llvm-cov report --lcov"
    DO rust+CARGO --args="llvm-cov --html --output-dir ./coverage"
    SAVE ARTIFACT ./lcov.info AS LOCAL lcov.info
    SAVE ARTIFACT ./coverage AS LOCAL coverage

crossbuild:
    ARG RUST_TARGET
    ARG BASE_IMAGE=ghcr.io/rust-cross/cargo-zigbuild:latest
    FROM +builder --IMAGE=$BASE_IMAGE
    RUN rustup target add $RUST_TARGET
    RUN cargo zigbuild --release --target $RUST_TARGET
    SAVE ARTIFACT ./target/$RUST_TARGET/release AS LOCAL ./target/$RUST_TARGET/release
    SAVE ARTIFACT ./target/$RUST_TARGET/release $RUST_TARGET

build:
    BUILD +crossbuild \
        --RUST_TARGET=x86_64-pc-windows-gnu \
        --RUST_TARGET=x86_64-unknown-linux-gnu \
        --RUST_TARGET=aarch64-unknown-linux-gnu \
        --RUST_TARGET=x86_64-apple-darwin \
        --RUST_TARGET=aarch64-apple-darwin

all:
	BUILD +fmt
	BUILD +lint
	BUILD +test
    BUILD +build
