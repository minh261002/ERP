# ---------- base ----------
FROM node:20-alpine AS base
WORKDIR /app
RUN corepack enable && corepack prepare pnpm@9.0.0 --activate

# ---------- deps ----------
FROM base AS deps
COPY package.json pnpm-lock.yaml ./
COPY apps/api/package.json apps/api/package.json
COPY packages ./packages
RUN pnpm install --frozen-lockfile

# ---------- build ----------
FROM deps AS builder
COPY . .
WORKDIR /app/apps/api

# Prisma needs schema at build time
COPY apps/api/prisma ./prisma

RUN pnpm build

# ---------- runner ----------
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copy compiled app
COPY --from=builder /app/apps/api/dist ./dist

# Prisma runtime needs schema
COPY --from=builder /app/apps/api/prisma ./prisma

# Only production deps (OPTIONAL but recommended)
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/apps/api/package.json ./package.json

EXPOSE 3000

# Prisma migrate deploy should be run externally (CI / entrypoint)
CMD ["node", "dist/main.js"]
