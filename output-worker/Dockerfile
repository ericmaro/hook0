# ---------------------------------------------------
# 1. Builder Stage
# ---------------------------------------------------
FROM rust:1.92-slim-bookworm AS builder-rust

# Install build dependencies
# protobuf-compiler is often needed for the shared libraries in the workspace
RUN apt-get update && apt-get install -y protobuf-compiler pkg-config libssl-dev

WORKDIR /app

# Copy the entire source code
COPY . .

# Build the Output Worker
# - We do NOT use "--no-default-features" here because the worker relies on the defaults.
# - We target the specific binary "hook0-output-worker".
RUN SQLX_OFFLINE=true cargo build --release --bin hook0-output-worker --locked

# ---------------------------------------------------
# 2. Runtime Stage
# ---------------------------------------------------
FROM debian:bookworm-slim

ARG UID=10001

# Install runtime dependencies
# "ca-certificates" is CRITICAL here because the worker sends HTTPS webhooks to external URLs.
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

# Copy the binary from builder
COPY --from=builder-rust /app/target/release/hook0-output-worker /hook0-output-worker

# Start the worker
CMD ["/hook0-output-worker"]
