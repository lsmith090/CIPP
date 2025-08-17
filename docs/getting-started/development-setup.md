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
# Clone the repository (replace with your fork's URL if contributing)
git clone https://github.com/[your-username]/CIPP.git
cd CIPP

# Navigate to frontend directory
cd CIPP
```

> **Note**: If you're contributing to CIPP, fork the repository first and clone your fork. If you're just setting up for local development, you can clone the main repository directly.

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

Example configuration for local development:
```env
# API Configuration - Point to your local CIPP-API instance
NEXT_PUBLIC_API_URL=http://localhost:7071

# Or point to an existing CIPP-API deployment for testing
# NEXT_PUBLIC_API_URL=https://your-existing-api.azurewebsites.net

# Azure AD Configuration (required for authentication)
# These values come from your Azure App Registration
NEXT_PUBLIC_CLIENT_ID=your-azure-app-client-id

# Development flags
NEXT_PUBLIC_DEV_MODE=true
NEXT_PUBLIC_DEBUG_MODE=true

# Feature flags (optional)
NEXT_PUBLIC_ENABLE_ANALYTICS=false
```

#### Production Environment (.env.production)
```env
# Production API endpoint
NEXT_PUBLIC_API_URL=https://your-api-domain.azurewebsites.net

# Production Azure AD client ID
NEXT_PUBLIC_CLIENT_ID=your-production-client-id

# Production flags
NEXT_PUBLIC_DEV_MODE=false
NEXT_PUBLIC_DEBUG_MODE=false
NEXT_PUBLIC_ENABLE_ANALYTICS=true
```

> **Important**: Never commit `.env.local` or any environment files containing secrets to version control. These files are included in `.gitignore` for security.

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

## Azure AD Setup (Required for Authentication)

CIPP requires Azure Active Directory configuration for authentication. You'll need to create an Azure App Registration:

### 1. Create Azure App Registration
1. Go to [Azure Portal](https://portal.azure.com) → Azure Active Directory → App registrations
2. Click "New registration"
3. Set name (e.g., "CIPP Development")
4. Set redirect URI: `http://127.0.0.1:3000/.auth/login/aad/callback`
5. Note the **Application (client) ID** for your `.env.local` file

### 2. Configure API Permissions
Add the following Microsoft Graph permissions:
- `User.Read` (default)
- `Directory.Read.All`
- Additional permissions as needed for your development scenario

### 3. Update Environment Variables
Add your client ID to `.env.local`:
```env
NEXT_PUBLIC_CLIENT_ID=your-application-client-id-here
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

# Check that the build works before committing
npm run build
```

### 3. Testing and Quality Assurance

CIPP uses manual testing and code quality tools:

```bash
# Run ESLint to check code quality
npm run lint

# Auto-fix linting issues where possible
npm run lint-fix
```

> **Note**: CIPP does not have automated unit or end-to-end tests configured. Testing is done through:
> - Manual testing of features during development
> - Code review process
> - ESLint for code quality enforcement
> 
> The project follows manual testing practices with ESLint for code quality.

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

#### "ESLint errors"
```bash
# Check for linting issues
npm run lint

# Automatically fix what can be fixed
npm run lint-fix

# For build errors, check the console output carefully
npm run build
```

#### "Build fails on deployment"
```bash
# Test production build locally
npm run build
npm run start

# Check for environment-specific issues
npm run export
```

## Getting Help

If you encounter issues during setup:

1. **Check the troubleshooting section** above for common problems
2. **Review your environment configuration** - many issues stem from incorrect `.env.local` setup
3. **Verify Node.js version** - CIPP requires Node.js ^22.13.0
4. **Check GitHub Issues** - Someone may have encountered the same problem
5. **Ask for help** - Create a GitHub issue or start a discussion

## Contributing Back

Once you have your development environment working:

1. **Test your setup** by making a small change and verifying it works
2. **Read the contribution guidelines** in the main repository
3. **Start with small contributions** like documentation improvements or bug fixes
4. **Join the community** to learn about ongoing projects and how to help

## Next Steps

Your development environment is ready! Continue with:

- **[Project Structure Guide](./project-structure.md)** - Understand how the codebase is organized
- **Component Documentation** - Learn about CIPP's component library
- **API Documentation** - Understand how to integrate with the CIPP API

Welcome to CIPP development!