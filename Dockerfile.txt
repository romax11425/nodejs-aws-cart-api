# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy and install only package files first — changes rarely
COPY package*.json ./

# Install all dependencies (cacheable if package.json doesn't change)
RUN npm ci

# Copy source code — changes frequently
COPY . .

# Build app (after source code)
RUN npm run build

# Remove dev dependencies for production
RUN npm prune --omit=dev


# Production stage
FROM node:18-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

# Only need package.json for runtime metadata (not strictly needed)
COPY package*.json ./

# Copy final production app & modules from builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 4000

CMD ["node", "dist/main"]