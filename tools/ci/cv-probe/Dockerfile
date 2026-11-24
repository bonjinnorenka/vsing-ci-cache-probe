# syntax=docker/dockerfile:1.7
# TEMPORARY PROBE (delete after use): the smallest possible rustc compile
# through the real sccache 0.18.0 GHA backend, so a cache-version mismatch
# shows up as `Cache misses` / `wrote to cache` in the sccache log.
FROM rust:1.99.0-bookworm

ARG PROBE_RUNNER="none"

RUN --mount=type=secret,id=sccache_env,target=/run/secrets/sccache_env \
    set -eux; \
    curl -fsSL https://github.com/mozilla/sccache/releases/download/v0.18.0/sccache-v0.18.0-x86_64-unknown-linux-musl.tar.gz \
      | tar -xz --strip-components=1 -C /usr/local/bin --wildcards '*/sccache'; \
    sccache --version; \
    set -a && . /run/secrets/sccache_env && set +a; \
    export SCCACHE_SERVER_UDS=/run/sccache.sock SCCACHE_IDLE_TIMEOUT=0 \
      SCCACHE_NO_DAEMON=1 SCCACHE_ERROR_LOG=/tmp/sccache-error.log SCCACHE_LOG=info \
      SCCACHE_DIR=/sccache-l0 SCCACHE_CACHE_SIZE=1G; \
    mkdir -p /sccache-l0; \
    sccache --start-server; \
    echo "PROBE_SCCACHE runner=$PROBE_RUNNER"; \
    mkdir -p /probe/src; \
    printf '%s\n' 'pub fn probe_value() -> u64 { 42 }' 'fn main() { println!("{}", probe_value()); }' > /probe/src/main.rs; \
    cd /probe; \
    RUSTC_WRAPPER=/usr/local/bin/sccache rustc --edition 2021 --crate-type bin -o /probe_probe src/main.rs; \
    sccache --show-stats; \
    cat /tmp/sccache-error.log 2>/dev/null | tail -20 || true
