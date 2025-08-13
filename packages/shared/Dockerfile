# Use Node.js 20 as the base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package.json and yarn.lock for dependency installation
COPY package.json ./
COPY ../../yarn.lock ./
COPY ../../.yarnrc.yml ./

# Install dependencies using Yarn
RUN yarn install --frozen-lockfile

# Copy source code
COPY . .

# Build the TypeScript package
RUN yarn build

# Expose the built dist directory
VOLUME ["/app/dist"]

# Default command (can be overridden)
CMD ["echo", "Shared package built successfully"]
