# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Novu is an open-source notification infrastructure platform that provides a unified API for multi-channel notifications (Email, SMS, Push, Chat, In-App). It's built as a monorepo using Nx with TypeScript, featuring microservices architecture and modern development tooling.

## Key Commands

### Setup and Development
```bash
# Initial setup - installs dependencies and builds all packages
npm run setup:project

# Interactive development helper (recommended for new developers)
npm run start  # or npm run jarvis

# Environment setup (macOS/Linux)
npm run dev-environment-setup
```

### Building
```bash
# Build all packages (excludes nextjs,nestjs)
npm run build

# Build specific services
npm run build:api         # @novu/api-service
npm run build:dashboard   # @novu/dashboard  
npm run build:web         # @novu/web
npm run build:worker      # @novu/worker
npm run build:ws          # @novu/ws
npm run build:packages    # All publishable packages
```

### Running Services
```bash
# Start specific services
npm run start:api:dev     # API with hot reload (port 3000)
npm run start:dashboard   # Dashboard (port 4201)
npm run start:web         # Web app (port 4200)
npm run start:worker      # Background worker (port 3004)
npm run start:ws          # WebSocket service (port 3002)
```

### Testing
```bash
# Lint all projects
npm run lint

# Test providers
npm run test:providers

# E2E tests (from root)
npm run test:e2e:novu-v0  # Legacy API tests
npm run test:e2e:novu-v2  # Current API tests
```

### Application-specific Tests
```bash
# API tests (run from /apps/api/)
npm run test              # Unit tests
npm run test:e2e          # E2E tests

# Web/Dashboard tests (run from respective app dirs)
npm run test:e2e          # Playwright E2E tests
npm run test:e2e:ui       # Playwright with UI
npm run test:e2e:debug    # Debug mode
```

## Architecture

### Microservices (`/apps/`)
- **api** - Main NestJS API service (business logic, REST endpoints)
- **dashboard** - New React/Vite dashboard application  
- **web** - Legacy React web application
- **worker** - Background job processing service
- **ws** - WebSocket service for real-time features
- **webhook** - External webhook handling service
- **inbound-mail** - Email parsing and processing service

### Libraries (`/libs/`)
- **dal** - Data Access Layer (MongoDB/Redis abstractions)
- **application-generic** - Common utilities, health checks, encryption
- **design-system** - Shared UI components
- **novui** - UI component library with Panda CSS
- **notifications** - Core notification logic
- **testing** - Testing utilities and services

### Packages (`/packages/`)
Published NPM packages:
- **framework** - Core Novu Framework for notification workflows
- **js** - JavaScript SDK with UI components
- **react** - React-specific components and hooks
- **providers** - Communication provider integrations
- **shared** - Shared types and utilities

## Development Environment

### Prerequisites
- Node.js v20.19.0 (LTS) - Required version
- pnpm v10.0.0+ - Package manager
- MongoDB - Database
- Redis - Cache/Queue
- Docker - For local infrastructure

### Infrastructure Setup
```bash
# Minimal setup (MongoDB + Redis)
docker-compose -f ./docker-compose.minimal.yml up -d

# Full development infrastructure
docker-compose -f ./docker/local/docker-compose.yml up -d
```

### Service Ports
- API: http://127.0.0.1:3000
- Dashboard: http://127.0.0.1:4201
- Web: http://127.0.0.1:4200
- WebSocket: http://127.0.0.1:3002
- Worker: http://127.0.0.1:3004

## Code Style Guidelines

### General Conventions
- TypeScript for all code
- Functional programming patterns over classes
- Descriptive variable names with auxiliary verbs (isLoading, hasError)
- Use interfaces over types (backend), types over interfaces (frontend)
- Lowercase with dashes for directories/files (auth-wizard)
- Named exports for components

### Frontend Specific
- Use functional components with TypeScript
- Avoid nested ternaries
- Wrap client components in Suspense with fallback
- Use dynamic loading for non-critical components
- Structure: exported component, subcomponents, helpers, static content, types

### Import Rules
- Import motion components from "motion/react" (not "motion-react")
- Add blank lines before return statements

### Git Conventions
- Use proper scope in commit titles (dashboard, web, api, worker, shared, etc.)
- Main branch: `next`
- Current release branch: `release-2.3.0`

## Build System

- **Nx** - Monorepo build orchestration with caching
- **pnpm** - Package manager with workspace support
- **TypeScript** - Type safety across all projects
- **ESLint** - Code quality and linting

## Testing Strategy

- **Jest** - Unit testing framework (ts-jest preset)
- **Playwright** - E2E testing for web applications  
- **Mocha** - Integration testing for API services
- Tests located alongside source files (`*.spec.ts`, `*.e2e.ts`)

## Development Workflow

1. Use `npm run jarvis` for guided development setup
2. Start with minimal infrastructure via Docker Compose
3. Run specific services as needed for your work
4. Use hot reload modes for active development
5. Run tests frequently - `npm run lint` catches most issues
6. Follow the existing code patterns and conventions

## Enterprise Features

Enterprise code is located in:
- `/enterprise/` folder (root level)
- `/apps/web/src/ee/`
- `/apps/dashboard/src/ee/`

These require commercial licensing and are built by the core team.