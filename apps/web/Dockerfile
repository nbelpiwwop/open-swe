# Use Node.js 20 as the base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Enable Corepack to use the project's specified Yarn version
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

# Copy web app package files
COPY apps/web/package.json ./apps/web/
COPY apps/web/tsconfig.json ./apps/web/
COPY apps/web/next.config.mjs ./apps/web/
COPY apps/web/tailwind.config.js ./apps/web/
COPY apps/web/postcss.config.mjs ./apps/web/
COPY apps/web/components.json ./apps/web/
COPY apps/web/prettier.config.js ./apps/web/

# Install dependencies
RUN yarn install --frozen-lockfile

# Build the shared package dependency first
WORKDIR /app/packages/shared
RUN yarn build

# Copy web app source code
WORKDIR /app
COPY apps/web/src ./apps/web/src
COPY apps/web/public ./apps/web/public
COPY apps/web/eslint.config.js ./apps/web/

# Build the Next.js application for production
WORKDIR /app/apps/web
RUN yarn build

# Expose port 3000
EXPOSE 3000

# Command to start the production server
CMD ["yarn", "start"]

