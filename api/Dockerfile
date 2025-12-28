# ---------------------------------------------------
# 1. Builder Stage
# ---------------------------------------------------
FROM rust:1.92-slim-bookworm AS builder-rust

# Install build dependencies
RUN apt-get update && apt-get install -y protobuf-compiler pkg-config libssl-dev

WORKDIR /app

# Copy source code
COPY . .

# Build the API
# CHANGE: Added specific features that were previously in docker-compose
# - reqwest-rustls-tls-webpki-roots: Required for secure HTTP requests
# - application-secret-compatibility: Required for Hook0 logic
RUN SQLX_OFFLINE=true cargo build --release --bin hook0-api --locked --no-default-features --features "reqwest-rustls-tls-webpki-roots,application-secret-compatibility"

# ---------------------------------------------------
# 2. Runtime Stage
# ---------------------------------------------------
FROM debian:bookworm-slim

ARG UID=10001

# Install runtime dependencies
RUN apt-get update -y \
    && apt-get install -y --no-install-recommends adduser curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

USER appuser

# Copy the binary
COPY --from=builder-rust /app/target/release/hook0-api /hook0-api

ENV DISABLE_SERVING_WEBAPP=true

CMD ["/hook0-api"]
