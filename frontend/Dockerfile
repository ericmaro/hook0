# ---------------------------------------------------
# 1. Builder Stage
# ---------------------------------------------------
FROM node:22-slim AS build

WORKDIR /app

# Define the build argument for the API URL
ARG API_ENDPOINT="https://hook0-api.up.railway.app/api/v1"

# Expose it as an environment variable
ENV API_ENDPOINT=${API_ENDPOINT}

# 1. Copy the 'mediakit' folder to the root
COPY mediakit /mediakit

# 2. Install dependencies
COPY frontend/package.json frontend/package-lock.json ./

# CHANGE: Switched from 'npm ci' to 'npm install'
# 'npm ci' failed because the lockfile in the repo is out of sync.
# '--legacy-peer-deps' fixes the React 19 vs React 18 version conflict.
RUN npm install --legacy-peer-deps

# 3. Copy the rest of the frontend source code
COPY frontend/ .

# 4. Build the static site
RUN npm run build

# ---------------------------------------------------
# 2. Runtime Stage
# ---------------------------------------------------
FROM caddy:2-alpine

# Configure Caddy for SPA routing (prevents 404s on refresh)
RUN sed -i '/file_server/i try_files {path} /index.html' /etc/caddy/Caddyfile

# Add curl for healthchecks
RUN apk --no-cache add curl

# Copy the built assets from the builder stage
COPY --from=build /app/dist /usr/share/caddy/

EXPOSE 80
