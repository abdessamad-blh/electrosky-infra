# syntax=docker/dockerfile:1
# Mastra requires Node >= 22.13 (the other Electro Sky images use 20).

# ── Build ─────────────────────────────────────────────────────────────────────
FROM node:22-alpine AS build
WORKDIR /app
# No CLI usage analytics from build servers
ENV MASTRA_TELEMETRY_DISABLED=1

COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm npm ci

COPY tsconfig.json ./
COPY src ./src
# `mastra build` bundles the app into .mastra/output and runs its own
# `npm install` there (Linux/musl native binaries, e.g. libsql). The cache
# mount keeps that install fast on rebuilds.
RUN --mount=type=cache,target=/root/.npm npx mastra build

# ── Run ───────────────────────────────────────────────────────────────────────
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production \
    MASTRA_TELEMETRY_DISABLED=1 \
    HOST=0.0.0.0 \
    PORT=4111 \
    ASSISTANT_DB_URL=file:/app/data/assistant.db

COPY --from=build --chown=node:node /app/.mastra/output ./
RUN mkdir -p /app/data && chown node:node /app/data

USER node
EXPOSE 4111
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://127.0.0.1:4111/assistant/status >/dev/null || exit 1

CMD ["node", "index.mjs"]
