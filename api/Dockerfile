# ---------------------------------------------------
# 1. Builder Stage
# ---------------------------------------------------
FROM rust:1.92-slim-bookworm AS builder-rust

# Install dependencies required for building (Protobuf is needed for gRPC)
RUN apt-get update && apt-get install -y protobuf-compiler pkg-config libssl-dev

WORKDIR /app

# COPY the entire source code into the container
# This replaces the complex "--mount=type=bind" commands
COPY . .

# Build the API
# We use SQLX_OFFLINE=true to avoid needing a live DB connection during build
ARG FEATURES=""
RUN SQLX_OFFLINE=true cargo build --release --bin hook0-api --locked --no-default-features --features "${FEATURES}"

# ---------------------------------------------------
# 2. Runtime Stage
# ---------------------------------------------------
FROM debian:bookworm-slim

ARG UID=10001

# Install runtime dependencies (curl for healthchecks, ca-certificates for SSL)
RUN apt-get update -y \
    && apt-get install -y --no-install-recommends adduser curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Create a non-root user for security
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

USER appuser

# Copy the compiled binary from the builder stage
COPY --from=builder-rust /app/target/release/hook0-api /hook0-api

# Set environment variables
ENV DISABLE_SERVING_WEBAPP=true

# Start the application
CMD ["/hook0-api"]
