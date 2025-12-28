# ---------------------------------------------------
# 1. Builder Stage
# ---------------------------------------------------
# Use Node 22 (LTS) because Node 24 does not exist yet
FROM node:22-slim AS build

WORKDIR /app

# Define the build argument for the API URL
ARG API_ENDPOINT="https://hook0-api.up.railway.app/api/v1"

# Expose it as an environment variable so the build tool (Vite) can read it
ENV API_ENDPOINT=${API_ENDPOINT}

# 1. Copy the 'mediakit' folder to the root (previously handled by mount)
# This assumes 'mediakit' is in the root of your repo
COPY mediakit /mediakit

# 2. Install dependencies
# We copy package files first to optimize Docker caching
COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci

# 3. Copy the rest of the frontend source code
COPY frontend/ .

# 4. Build the static site
RUN npm run build

# ---------------------------------------------------
# 2. Runtime Stage
# ---------------------------------------------------
# Use the stable Caddy 2 image
FROM caddy:2-alpine

# Configure Caddy to handle Single Page Application (SPA) routing
# This ensures that refreshing a sub-page (like /dashboard) doesn't give a 404
RUN sed -i '/file_server/i try_files {path} /index.html' /etc/caddy/Caddyfile

# Add curl for healthchecks
RUN apk --no-cache add curl

# Copy the built assets from the builder stage
COPY --from=build /app/dist /usr/share/caddy/

EXPOSE 80
