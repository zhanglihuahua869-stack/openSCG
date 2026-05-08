# Stage 1: Build
FROM node:22-alpine AS builder

WORKDIR /app

RUN corepack enable

# In staging context, we are at the root of openscg_app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

# Install all dependencies (including dev)
ENV PUPPETEER_SKIP_DOWNLOAD="true"
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD="true"
ENV CI="true"
RUN pnpm install --frozen-lockfile --reporter=append-only --config.strict-dep-builds=false

# Copy source and public folder
COPY . .

# Build Next.js with NO_REDIS flag to avoid connection errors during static generation
ENV NO_REDIS="true"
RUN pnpm --config.verify-deps-before-run=false run build

# Stage 2: Production
FROM node:22-alpine AS runner

WORKDIR /app

RUN corepack enable

ENV NODE_ENV=production
# Next.js 15 listens on 3000 by default, Cloud Run provides PORT.
# We set HOSTNAME to ensure it listens externally.
ENV HOSTNAME="0.0.0.0"

# Copy build artifacts and necessary files from builder
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./
COPY --from=builder /app/pnpm-lock.yaml ./
COPY --from=builder /app/pnpm-workspace.yaml ./
COPY --from=builder /app/server.js ./
COPY --from=builder /app/next.config.mjs ./
# We need node_modules for server.js to run (socket.io etc)
# Using production-only install in runner to minimize image size
ENV PUPPETEER_SKIP_DOWNLOAD="true"
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD="true"
ENV CI="true"
RUN pnpm install --prod --frozen-lockfile --reporter=append-only --config.strict-dep-builds=false

EXPOSE 3000

# Using node server.js because it's our custom socket.io server
CMD ["node", "server.js"]
