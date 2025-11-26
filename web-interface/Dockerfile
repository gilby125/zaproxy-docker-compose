# Build stage
FROM node:20-alpine AS builder

RUN npm install -g pnpm
WORKDIR /app

# Clone and build
RUN apk add --no-cache git
RUN git clone https://github.com/rpuls/ZAP-OWASP-Web-interface.git .
RUN pnpm install
RUN pnpm build

# Production stage
FROM node:20-alpine

RUN npm install -g pnpm
WORKDIR /app

# Copy built application
COPY --from=builder /app /app

EXPOSE 3001

CMD ["pnpm", "start"]
