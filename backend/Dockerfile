# Multi-stage Dockerfile: build frontend with Node, then build backend Python image.
# Place this file at backend/Dockerfile

# --- Stage 1: frontend build (optional, only if frontend/ exists) ---
FROM node:18-alpine AS frontend-build
WORKDIR /frontend
# Copy only package files first for better caching
COPY frontend/package*.json frontend/ ./
# If you use pnpm/yarn, adjust commands accordingly
RUN if [ -f package.json ]; then npm ci; fi
# Copy full frontend sources and build
COPY frontend/ ./ 
RUN if [ -f package.json ]; then npm run build; fi

# --- Stage 2: backend (python) ---
FROM python:3.11-slim

# Install build deps for psycopg2
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc libpq-dev build-essential curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy backend requirements and install
COPY backend/requirements.txt ./requirements.txt
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Copy backend source
COPY backend/ ./

# Copy built frontend assets into backend static folder (adjust path if necessary)
# This assumes frontend build output is at frontend/build or frontend/dist
COPY --from=frontend-build /frontend/build ./frontend_build
# If your backend expects assets in a different location, update the copy target above.

ENV PYTHONUNBUFFERED=1
ENV WEB_CONCURRENCY=2

EXPOSE 8000

# Use gunicorn with uvicorn workers; Render will set $PORT at runtime.
CMD ["sh", "-c", "exec gunicorn -k uvicorn.workers.UvicornWorker app.main:app --bind 0.0.0.0:${PORT:-8000} --workers ${WEB_CONCURRENCY} --log-level info"]
