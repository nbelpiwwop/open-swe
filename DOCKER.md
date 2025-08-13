# Docker Setup Guide for Open-SWE

This guide provides comprehensive instructions for setting up and running Open-SWE using Docker Compose for self-hosting.

## Prerequisites

Before starting, ensure you have the following installed:

- **Docker** (version 20.10 or higher)
- **Docker Compose** (version 2.0 or higher)
- **Git** (for cloning the repository)

## Architecture Overview

Open-SWE consists of three main services:

- **`shared`**: Builds the shared utilities package that other services depend on
- **`agent`**: LangGraph agent service running on port 2024
- **`web`**: Next.js web application running on port 3000

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/langchain-ai/open-swe.git
cd open-swe
```

### 2. Set Up Environment Files

Copy the Docker environment templates and configure them:

```bash
# Copy agent environment file
cp apps/open-swe/docker.env.example apps/open-swe/.env

# Copy web app environment file
cp apps/web/docker.env.example apps/web/.env
```

### 3. Configure Environment Variables

Edit the environment files with your specific configuration:

#### Agent Service (`apps/open-swe/.env`)

**Required Variables:**
```bash
# LLM Provider Keys (at least one required)
ANTHROPIC_API_KEY="your_anthropic_key_here"
OPENAI_API_KEY="your_openai_key_here"
GOOGLE_API_KEY="your_google_key_here"

# GitHub App Configuration
GITHUB_APP_NAME="your-github-app-name"
GITHUB_APP_ID="your_github_app_id"
GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
your_private_key_here
-----END RSA PRIVATE KEY-----"
GITHUB_WEBHOOK_SECRET="your_webhook_secret"

# Encryption Key (generate with: openssl rand -hex 32)
SECRETS_ENCRYPTION_KEY="your_32_byte_hex_key_here"
```

**Optional Variables:**
```bash
# LangSmith Tracing
LANGCHAIN_API_KEY="lsv2_pt_your_key_here"
LANGCHAIN_PROJECT="your_project_name"

# Infrastructure
DAYTONA_API_KEY="your_daytona_key"
FIRECRAWL_API_KEY="your_firecrawl_key"
```

#### Web Service (`apps/web/.env`)

**Required Variables:**
```bash
# GitHub OAuth Configuration
NEXT_PUBLIC_GITHUB_APP_CLIENT_ID="your_github_client_id"
GITHUB_APP_CLIENT_SECRET="your_github_client_secret"
GITHUB_APP_REDIRECT_URI="http://localhost:3000/api/auth/github/callback"

# GitHub App Configuration (must match agent service)
GITHUB_APP_NAME="your-github-app-name"
GITHUB_APP_ID="your_github_app_id"
GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
your_private_key_here
-----END RSA PRIVATE KEY-----"

# Encryption Key (must match agent service)
SECRETS_ENCRYPTION_KEY="your_32_byte_hex_key_here"
```

### 4. Build and Run Services

Build and start all services:

```bash
docker-compose up --build
```

Or run in detached mode:

```bash
docker-compose up --build -d
```

### 5. Access the Application

- **Web Interface**: http://localhost:3000
- **LangGraph Agent API**: http://localhost:2024

## Detailed Configuration

### GitHub App Setup

You'll need to create a GitHub App for authentication and repository access:

1. Go to GitHub Settings → Developer settings → GitHub Apps
2. Click "New GitHub App"
3. Configure the app with the following settings:
   - **App name**: Choose a unique name (use this for `GITHUB_APP_NAME`)
   - **Homepage URL**: `http://localhost:3000` (for development)
   - **Callback URL**: `http://localhost:3000/api/auth/github/callback`
   - **Webhook URL**: `http://localhost:2024/webhooks` (if using webhooks)
   - **Permissions**: Repository permissions as needed
4. Generate a private key and use it for `GITHUB_APP_PRIVATE_KEY`
5. Note the App ID for `GITHUB_APP_ID`
6. Generate a client secret for `GITHUB_APP_CLIENT_SECRET`

### Environment Variable Reference

#### Critical Variables

These variables must be configured for the system to work:

| Variable | Service | Description |
|----------|---------|-------------|
| `ANTHROPIC_API_KEY` | Agent | Anthropic API key (recommended) |
| `GITHUB_APP_NAME` | Both | GitHub App name (must match) |
| `GITHUB_APP_ID` | Both | GitHub App ID |
| `GITHUB_APP_PRIVATE_KEY` | Both | GitHub App private key |
| `SECRETS_ENCRYPTION_KEY` | Both | 32-byte hex encryption key (must match) |
| `NEXT_PUBLIC_GITHUB_APP_CLIENT_ID` | Web | GitHub App client ID |
| `GITHUB_APP_CLIENT_SECRET` | Web | GitHub App client secret |

#### Docker-Specific Networking

The following variables are pre-configured for Docker Compose networking:

- `LANGGRAPH_API_URL=http://agent:2024` (web service connects to agent)
- `OPEN_SWE_APP_URL=http://web:3000` (agent connects to web service)
- `NEXT_PUBLIC_API_URL=http://localhost:3000/api` (client-side API calls)

## Docker Commands

### Basic Operations

```bash
# Build and start all services
docker-compose up --build

# Start services (without rebuilding)
docker-compose up

# Start in detached mode
docker-compose up -d

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# View logs for specific service
docker-compose logs agent
docker-compose logs web
```

### Development Commands

```bash
# Rebuild specific service
docker-compose build agent
docker-compose build web

# Restart specific service
docker-compose restart agent
docker-compose restart web

# Execute commands in running container
docker-compose exec agent bash
docker-compose exec web bash

# View service status
docker-compose ps
```

### Cleanup Commands

```bash
# Remove stopped containers
docker-compose rm

# Remove all containers, networks, and volumes
docker-compose down --volumes --remove-orphans

# Remove all images
docker-compose down --rmi all

# Full cleanup (containers, networks, volumes, images)
docker-compose down --volumes --remove-orphans --rmi all
```

## Troubleshooting

### Common Issues

#### 1. Port Already in Use

**Error**: `Port 3000 is already allocated` or `Port 2024 is already allocated`

**Solution**: 
```bash
# Check what's using the port
lsof -i :3000
lsof -i :2024

# Kill the process or change ports in docker-compose.yml
# Example: Change to different ports
ports:
  - "3001:3000"  # Web service
  - "2025:2024"  # Agent service
```

#### 2. Environment Variables Not Loading

**Error**: Missing API keys or configuration errors

**Solution**:
- Ensure `.env` files exist in the correct locations
- Check file permissions: `chmod 644 apps/open-swe/.env apps/web/.env`
- Verify environment file paths in `docker-compose.yml`
- Restart services after changing environment files

#### 3. Build Failures

**Error**: Docker build fails or dependency issues

**Solution**:
```bash
# Clean build (remove cache)
docker-compose build --no-cache

# Remove all containers and rebuild
docker-compose down
docker system prune -f
docker-compose up --build
```

#### 4. Service Communication Issues

**Error**: Web app can't connect to agent service

**Solution**:
- Verify `LANGGRAPH_API_URL=http://agent:2024` in web service
- Check that both services are on the same network
- Ensure agent service is healthy: `docker-compose logs agent`

#### 5. GitHub Authentication Issues

**Error**: OAuth login fails or GitHub API errors

**Solution**:
- Verify GitHub App configuration matches environment variables
- Check redirect URI matches: `http://localhost:3000/api/auth/github/callback`
- Ensure GitHub App has correct permissions
- Verify client ID and secret are correct

### Debugging Tips

#### Check Service Health

```bash
# View all service logs
docker-compose logs -f

# Check specific service logs
docker-compose logs -f agent
docker-compose logs -f web

# Check service status
docker-compose ps
```

#### Inspect Service Configuration

```bash
# View environment variables in running container
docker-compose exec agent env
docker-compose exec web env

# Access container shell
docker-compose exec agent sh
docker-compose exec web sh
```

#### Network Debugging

```bash
# Test inter-service communication
docker-compose exec web curl http://agent:2024/health
docker-compose exec agent curl http://web:3000

# Inspect Docker networks
docker network ls
docker network inspect open-swe-network
```

### Performance Optimization

#### Resource Limits

Add resource limits to `docker-compose.yml`:

```yaml
services:
  agent:
    # ... other configuration
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '1.0'
        reservations:
          memory: 1G
          cpus: '0.5'
```

#### Build Optimization

```bash
# Use BuildKit for faster builds
DOCKER_BUILDKIT=1 docker-compose build

# Parallel builds
docker-compose build --parallel
```

## Production Deployment

### Security Considerations

1. **Environment Variables**: Use Docker secrets or external secret management
2. **Network Security**: Use reverse proxy (nginx) for HTTPS
3. **Resource Limits**: Set appropriate CPU/memory limits
4. **Updates**: Implement proper update strategy with health checks

### Production Configuration

```yaml
# docker-compose.prod.yml example
version: '3.8'
services:
  web:
    environment:
      - GITHUB_APP_REDIRECT_URI=https://yourdomain.com/api/auth/github/callback
      - NEXT_PUBLIC_API_URL=https://yourdomain.com/api
    restart: always
    
  agent:
    environment:
      - OPEN_SWE_APP_URL=https://yourdomain.com
    restart: always
```

## Support

If you encounter issues not covered in this guide:

1. Check the [main documentation](https://docs.langchain.com/labs/swe/)
2. Review Docker Compose logs for error details
3. Ensure all required environment variables are configured
4. Verify GitHub App setup and permissions

For additional help, refer to the project's GitHub repository or community forums.
