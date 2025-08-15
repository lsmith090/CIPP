# Development Setup Guide

This guide will help you set up a complete development environment for the CIPP frontend application.

## Prerequisites

### Required Software

#### Node.js and Package Manager
```bash
# Required Node.js version
node: ^22.13.0

# Verify installation
node --version
npm --version
```

#### Git
```bash
# Verify Git installation
git --version
```

### Recommended Tools

#### Code Editor
- **Visual Studio Code** with extensions:
  - ES7+ React/Redux/React-Native snippets
  - Prettier - Code formatter
  - ESLint
  - Material-UI Snippets
  - TypeScript Hero

#### Browser Extensions
- React Developer Tools
- Redux DevTools
- Lighthouse (for performance auditing)

## Project Setup

### 1. Clone the Repository

```bash
# Clone the main repository
git clone https://github.com/KelvinTegelaar/CIPP.git
cd CIPP

# Navigate to frontend directory
cd CIPP
```

### 2. Install Dependencies

```bash
# Install all dependencies
npm install

# Verify installation
npm list --depth=0
```

### 3. Environment Configuration

Create environment files for different stages:

#### Development Environment (.env.local)
```bash
# Create local environment file
touch .env.local
```

Example configuration:
```env
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:7071

# Authentication
NEXT_PUBLIC_CLIENT_ID=your-azure-app-client-id

# Development flags
NEXT_PUBLIC_DEV_MODE=true
NEXT_PUBLIC_DEBUG_MODE=true

# Feature flags
NEXT_PUBLIC_ENABLE_ANALYTICS=false
```

#### Production Environment (.env.production)
```env
# Production API endpoint
NEXT_PUBLIC_API_URL=https://your-api-domain.azurewebsites.net

# Production client ID
NEXT_PUBLIC_CLIENT_ID=your-production-client-id

# Production flags
NEXT_PUBLIC_DEV_MODE=false
NEXT_PUBLIC_DEBUG_MODE=false
NEXT_PUBLIC_ENABLE_ANALYTICS=true
```

## Development Commands

### Available Scripts

```bash
# Start development server (binds to 127.0.0.1:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Static export for Azure Static Web Apps
npm run export

# Code linting
npm run lint

# Auto-fix linting issues
npm run lint-fix

# Start with Static Web App CLI (for Azure development)
npm run start-swa
```

### Development Server

Start the development server:
```bash
npm run dev
```

The application will be available at:
- **Frontend**: http://127.0.0.1:3000
- **API** (if running locally): http://localhost:7071

#### Development Features
- **Hot Reloading** - Automatic page refresh on file changes
- **Fast Refresh** - Component state preservation during updates
- **Source Maps** - Debug with original source code
- **Error Overlay** - In-browser error display

## Backend Integration Setup

### Option 1: Local API Development

#### Prerequisites
- PowerShell 7+
- Azure Functions Core Tools v4
- Docker Desktop (for storage emulation)

#### Setup Steps
```bash
# Navigate to API directory
cd ../CIPP-API

# Install PowerShell modules
./Tools/Initialize-DevEnvironment.ps1

# Start local development with Docker
docker-compose up

# Alternative: Start with Azure Functions Core Tools
func start
```

### Option 2: Connect to Existing API

Update your `.env.local` to point to an existing API instance:
```env
NEXT_PUBLIC_API_URL=https://your-existing-api.azurewebsites.net
```

## Azure Static Web Apps Development

For full Azure SWA development experience:

### 1. Install SWA CLI
```bash
npm install -g @azure/static-web-apps-cli
```

### 2. Configure SWA
Create or update `.vscode/settings.json`:
```json
{
  "azureStaticWebApps.showGettingStarted": false,
  "azureStaticWebApps.defaultPort": "3000",
  "azureStaticWebApps.apiPort": "7071"
}
```

### 3. Start SWA Development
```bash
npm run start-swa
```

This provides:
- Authentication simulation
- API proxy configuration
- Routing rule testing
- Environment variable injection

## Development Workflow

### 1. Feature Development

```bash
# Create feature branch
git checkout -b feature/your-feature-name

# Start development server
npm run dev

# Make changes and test
# Frontend runs on http://127.0.0.1:3000
```

### 2. Code Quality Checks

```bash
# Run linting
npm run lint

# Fix auto-fixable issues
npm run lint-fix

# Type checking (if using TypeScript)
npx tsc --noEmit
```

### 3. Testing

```bash
# Run unit tests (when available)
npm test

# Run e2e tests (when available)
npm run test:e2e
```

### 4. Build Verification

```bash
# Test production build
npm run build
npm run start

# Test static export
npm run export
```

## Debugging and Development Tools

### Browser DevTools

#### React Developer Tools
- Install React DevTools browser extension
- Access via browser DevTools → React tab
- Inspect component hierarchy and props
- Profile component performance

#### Redux DevTools
- Install Redux DevTools browser extension
- Monitor state changes and actions
- Time travel debugging
- Action replay and dispatch

### VS Code Configuration

#### Recommended Settings (.vscode/settings.json)
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "javascript.preferences.quoteStyle": "single",
  "typescript.preferences.quoteStyle": "single",
  "emmet.includeLanguages": {
    "javascript": "javascriptreact"
  }
}
```

#### Launch Configuration (.vscode/launch.json)
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js: debug server-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev"
    },
    {
      "name": "Next.js: debug client-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev",
      "serverReadyAction": {
        "pattern": "started server on .+, url: (https?://.+)",
        "uriFormat": "%s",
        "action": "debugWithChrome"
      }
    }
  ]
}
```

## Common Development Issues

### Port Conflicts
If port 3000 is in use:
```bash
# Use different port
npm run dev -- -p 3001

# Or set in environment
PORT=3001 npm run dev
```

### Memory Issues
For large projects:
```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max_old_space_size=4096" npm run dev
```

### Module Resolution Issues
```bash
# Clear Next.js cache
rm -rf .next

# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

### API Connection Issues

#### CORS Issues
Ensure your API includes the frontend URL in CORS settings:
```javascript
// In your API configuration
cors: {
  origin: ['http://127.0.0.1:3000', 'http://localhost:3000'],
  credentials: true
}
```

#### Authentication Issues
Check Azure AD app registration:
1. Redirect URIs include your development URL
2. API permissions are properly configured
3. Client secret is valid (for server-side flows)

## Performance Optimization

### Development Performance
```bash
# Analyze bundle size
npm install -g @next/bundle-analyzer
ANALYZE=true npm run build
```

### Memory Profiling
```bash
# Start with memory profiling
NODE_OPTIONS="--inspect" npm run dev
```

Then open Chrome DevTools → Memory tab for heap snapshots.

## Next Steps

Once your development environment is set up:

1. [Review Project Structure](./project-structure.md)
2. [Explore Component Architecture](../architecture/components.md)
3. [Learn Page Patterns](../architecture/page-patterns.md)
4. [Understand API Integration](../guides/api-integration.md)

## Troubleshooting

### Common Error Messages

#### "Module not found"
```bash
# Ensure all dependencies are installed
npm install

# Check for case sensitivity issues
# Verify import paths match actual file names
```

#### "TypeScript errors"
```bash
# Generate or update type definitions
npm run type-check

# Skip type checking temporarily
SKIP_TYPE_CHECK=true npm run build
```

#### "Build fails on deployment"
```bash
# Test production build locally
npm run build
npm run start

# Check for environment-specific issues
npm run export
```

For additional help, consult the main CIPP documentation or create an issue in the GitHub repository.