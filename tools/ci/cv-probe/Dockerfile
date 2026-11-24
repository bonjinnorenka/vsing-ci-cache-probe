# syntax=docker/dockerfile:1.7
# TEMPORARY PROBE: minimal rustc compile through the real sccache 0.18.0
# GHA backend so a cache-version mismatch shows as `Cache misses` today and
# `Cache hits` once the slot-independent path is in effect.
FROM rust:1.99.0-bookworm

ARG PROBE_RUNNER="none"
ARG SCCACHE_GHA_VERSION_VALUE="cvprobe-v1"

RUN --mount=type=secret,id=sccache_env,target=/run/secrets/sccache_env \
    set -eux; \
    curl -fsSL https://github.com/mozilla/sccache/releases/download/v0.18.0/sccache-v0.18.0-x86_64-unknown-linux-musl.tar.gz \
      | tar -xz --strip-components=1 -C /usr/local/bin --wildcards '*/sccache'; \
    sccache --version; \
    set -a && . /run/secrets/sccache_env && set +a; \
    export SCCACHE_GHA_VERSION="$SCCACHE_GHA_VERSION_VALUE"; \
    export SCCACHE_SERVER_UDS=/run/sccache.sock SCCACHE_IDLE_TIMEOUT=0 \
      SCCACHE_NO_DAEMON=1 SCCACHE_ERROR_LOG=/tmp/sccache-error.log SCCACHE_LOG=info \
      SCCACHE_DIR=/sccache-l0 SCCACHE_CACHE_SIZE=1G; \
    mkdir -p /sccache-l0; \
    sccache --start-server; \
    cat "$COLAB_PROBE_PATH_FILE" > /tmp/probe_cwd; \
    mkdir -p "$(cat /tmp/probe_cwd)"; \
    printf '%s\n' 'pub fn probe_value() -> u64 { 42 }' 'fn main() { println!("{}", probe_value()); }' > "$(cat /tmp/probe_cwd)/main.rs"; \
    cd "$(cat /tmp/probe_cwd)"; \
    echo "PROBE_SCCACHE runner=$PROBE_RUNNER version=$SCCACHE_GHA_VERSION cwd=$(pwd)"; \
    RUSTC_WRAPPER=/usr/local/bin/sccache rustc --edition 2021 --crate-type bin -o /probe_probe main.rs; \
    sccache --show-stats; \
    cat /tmp/sccache-error.log 2>/dev/null | tail -20 || true
