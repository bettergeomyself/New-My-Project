#####################################################################
# Dockerfile.production
# Multi-stage production Dockerfile
# Node.js + Nginx + PM2 + Security Hardening
#####################################################################

############################
# STAGE 1 - BASE
############################
FROM node:20-alpine AS base

LABEL maintainer="yourname@example.com"
LABEL version="1.0.0"
LABEL description="Production Node.js App with Nginx and PM2"

ENV NODE_ENV=production
ENV APP_HOME=/usr/src/app

WORKDIR $APP_HOME

############################
# Install system packages
############################
RUN apk update && apk upgrade && \
    apk add --no-cache \
    bash \
    curl \
    wget \
    git \
    nginx \
    tini \
    dumb-init \
    ca-certificates \
    openssl && \
    rm -rf /var/cache/apk/*

############################
# Create non-root user
############################
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

############################
# STAGE 2 - DEPENDENCIES
############################
FROM base AS dependencies

COPY package*.json ./

RUN npm install --production && \
    npm cache clean --force

############################
# STAGE 3 - BUILD
############################
FROM base AS build

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

############################
# STAGE 4 - FINAL
############################
FROM node:20-alpine AS final

ENV NODE_ENV=production
ENV APP_HOME=/usr/src/app

WORKDIR $APP_HOME

########################################
# Install runtime dependencies only
########################################
RUN apk add --no-cache \
    nginx \
    curl \
    bash \
    tini \
    ca-certificates && \
    rm -rf /var/cache/apk/*

########################################
# Create user again
########################################
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

########################################
# Copy node_modules from dependencies
########################################
COPY --from=dependencies /usr/src/app/node_modules ./node_modules

########################################
# Copy built app
########################################
COPY --from=build /usr/src/app/dist ./dist
COPY --from=build /usr/src/app/package*.json ./

########################################
# Install PM2 globally
########################################
RUN npm install -g pm2 && \
    pm2 --version

########################################
# NGINX CONFIGURATION
########################################
RUN mkdir -p /run/nginx

COPY nginx.conf /etc/nginx/nginx.conf

########################################
# Permissions
########################################
RUN chown -R appuser:appgroup $APP_HOME && \
    chmod -R 755 $APP_HOME

########################################
# Security Hardening
########################################
RUN rm -rf /tmp/* \
    /var/tmp/* \
    /root/.npm \
    /root/.cache

########################################
# Expose Ports
########################################
EXPOSE 3000
EXPOSE 80

########################################
# Healthcheck
########################################
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD curl -f http://localhost:3000/health || exit 1

########################################
# Switch to non-root user
########################################
USER appuser

########################################
# Start with Tini
########################################
ENTRYPOINT ["/sbin/tini", "--"]

########################################
# Default CMD
########################################
CMD ["pm2-runtime", "dist/server.js"]

#####################################################################
# OPTIONAL ENV VARIABLES
#####################################################################

ENV PORT=3000
ENV LOG_LEVEL=info
ENV TZ=Asia/Ho_Chi_Minh

#####################################################################
# Extra Configuration Section
#####################################################################

# Logging setup
RUN mkdir -p /var/log/app && \
    chown -R appuser:appgroup /var/log/app

# Temp directory
RUN mkdir -p /usr/src/app/tmp && \
    chown -R appuser:appgroup /usr/src/app/tmp

# Cache directory
RUN mkdir -p /usr/src/app/cache && \
    chown -R appuser:appgroup /usr/src/app/cache

# Static files
RUN mkdir -p /usr/src/app/public && \
    chown -R appuser:appgroup /usr/src/app/public

#####################################################################
# Final metadata
#####################################################################

LABEL org.opencontainers.image.title="Node Production App"
LABEL org.opencontainers.image.description="Production-ready Docker Image"
LABEL org.opencontainers.image.authors="Your Name"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.source="https://github.com/yourrepo"

#####################################################################
# End of Dockerfile
#####################################################################
