# ==========================================
# Stage 1: Build
# ==========================================

FROM node:22-bookworm-slim AS builder

WORKDIR /app



RUN apt-get update \
    && apt-get install -y --no-install-recommends openssl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

    
COPY package.json package-lock.json ./

RUN npm ci

COPY . .

# Generate Prisma Client
RUN npx prisma generate

# Build NestJS
RUN npm run build

# Verify compiled entry point
RUN test -f dist/main.js


# ==========================================
# Stage 2: Production
# ==========================================

FROM node:22-bookworm-slim AS production

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=5000

# Install production dependencies

RUN apt-get update \
    && apt-get install -y --no-install-recommends openssl ca-certificates \
    && rm -rf /var/lib/apt/lists/*


COPY package.json package-lock.json ./


RUN npm ci --omit=dev \
    && npm cache clean --force

# Copy compiled application
COPY --from=builder /app/dist ./dist

# Copy Prisma files
COPY --from=builder /app/prisma ./prisma

# Copy generated Prisma Client
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma

# Verify entry point
RUN test -f dist/main.js

EXPOSE 5000

CMD ["node", "dist/main.js"]
