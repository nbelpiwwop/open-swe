# Use Node.js 20 as the base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Enable Corepack to use the correct Yarn version
RUN corepack enable

# Copy workspace configuration files
COPY package.json yarn.lock .yarnrc.yml ./

# Copy shared package files
COPY packages/shared/package.json ./packages/shared/
COPY packages/shared/tsconfig.json ./packages/shared/
COPY packages/shared/turbo.json ./packages/shared/

# Install dependencies using Yarn
RUN yarn install --frozen-lockfile

# Copy shared package source code
COPY packages/shared/src ./packages/shared/src
COPY packages/shared/eslint.config.js ./packages/shared/
COPY packages/shared/jest.config.js ./packages/shared/

# Build the shared package
WORKDIR /app/packages/shared
RUN yarn build

# Expose the built dist directory for other services to consume
VOLUME ["/app/packages/shared/dist"]

# Default command
CMD ["echo", "Shared package built successfully"]


