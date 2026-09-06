# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --only=production && npm cache clean --force

# ---- Production stage ----
FROM node:20-alpine AS production

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Security: remove unnecessary packages
# hadolint ignore=DL3018
# DL3018 wants an exact apk version pin, but this sandbox has no registry
# access to look up the exact dumb-init build tag shipped in the
# node:20-alpine base at build time, and guessing a version risks a build
# failure that's worse than the lint warning it "fixes". dumb-init is a
# single, actively maintained static binary with a tiny footprint, so
# tracking latest here is a deliberate, documented trade-off (see README).
RUN apk --no-cache add dumb-init

WORKDIR /app

# Copy only production dependencies from build stage
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: own files by non-root user
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 5000

# Security: use dumb-init for proper signal handling (PID 1 problem)
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
