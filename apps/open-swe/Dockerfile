# Use Node.js 20 as the base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Enable Corepack to use the correct Yarn version
RUN corepack enable

# Copy workspace configuration files
COPY package.json yarn.lock .yarnrc.yml ./
COPY turbo.json ./
COPY tsconfig.json ./

# Copy shared package files for dependency building
COPY packages/shared/package.json ./packages/shared/
COPY packages/shared/tsconfig.json ./packages/shared/
COPY packages/shared/turbo.json ./packages/shared/
COPY packages/shared/src ./packages/shared/src
COPY packages/shared/eslint.config.js ./packages/shared/
COPY packages/shared/jest.config.js ./packages/shared/

# Copy agent package files
COPY apps/open-swe/package.json ./apps/open-swe/
COPY apps/open-swe/tsconfig.json ./apps/open-swe/
COPY apps/open-swe/turbo.json ./apps/open-swe/

# Install all dependencies with yarn install
RUN yarn install --frozen-lockfile

# Build the shared package dependency first
WORKDIR /app/packages/shared
RUN yarn build

# Copy agent source code
WORKDIR /app
COPY apps/open-swe/src ./apps/open-swe/src
COPY apps/open-swe/eslint.config.js ./apps/open-swe/
COPY apps/open-swe/jest.config.js ./apps/open-swe/
COPY apps/open-swe/eval.tsconfig.json ./apps/open-swe/
COPY apps/open-swe/ls.vitest.config.ts ./apps/open-swe/
COPY apps/open-swe/scripts ./apps/open-swe/scripts

# Copy LangGraph configuration
COPY langgraph.json ./

# Build the TypeScript application
WORKDIR /app/apps/open-swe
RUN yarn build

# Expose port 2024
EXPOSE 2024

# Set working directory back to app root for langgraph command
WORKDIR /app

# Command to start the LangGraph development server
CMD ["yarn", "workspace", "@open-swe/agent", "dev"]

