# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a monorepo containing the Trippl travel planning platform with four main components:

- **api-server/**: Express.js backend API with TypeScript, PostgreSQL, Auth0 integration
- **app-web/**: Next.js 14 frontend application with React, Material-UI, Mapbox
- **aws-infrastructure/**: AWS CDK infrastructure as code for deployment
- **db-schema/**: Liquibase database schema management and migrations

## Development Commands

### API Server (api-server/)
```bash
# Local development
npm run start:local

# Linting and testing
npm run ci:lint
npm run ci:test

# Build
npm run build           # Windows
npm run ci:build        # CI/Linux

# Data import scripts
npm run import:local
npm run import:dev
npm run import:staging
npm run import:app

# Transform scripts for DMO data
npm run transform:sample
npm run transform:ca-pe-centralcoastal
# (see package.json for complete list)
```

### Web App (app-web/)
```bash
# Local development
npm run start:local

# Build for different environments
npm run build           # localhost
npm run build:dev
npm run build:staging
npm run build:app

# Testing and linting
npm run ci:test
npm run ci:lint

# Mobile app (Capacitor)
npm run static && npx cap sync
npx cap open ios
npx cap open android
```

### Database (db-schema/)
```bash
# Start local database
docker-compose --env-file .env.local -f docker-compose.yml up -d trippl-db

# Run migrations
docker-compose --env-file .env.local run --rm liquibase update

# Rollback to tag
docker-compose --env-file .env.local run --rm liquibase rollback <tag>

# Stop database
docker-compose --env-file .env.local -f docker-compose.yml down -d trippl-db
```

### AWS Infrastructure (aws-infrastructure/)
```bash
# Build and test
npm run build
npm run test

# CDK deployment
cdk bootstrap -c config=dev|staging|app
cdk deploy PolicyStack -c config=dev|staging|app
cdk deploy BucketStack -c config=dev|staging|app
cdk deploy CloudfrontStack -c config=dev|staging|app
cdk deploy DmzStack -c config=dev|staging|app
cdk deploy ApiStack -c config=dev|staging|app
cdk deploy AppStack -c config=dev|staging|app
```

## Architecture Overview

### Backend Architecture
- **Express.js API** with TypeScript, follows RESTful patterns
- **Data Layer**: PostgreSQL with Sequelize ORM
- **Authentication**: Auth0 JWT with custom middleware
- **File Storage**: AWS S3 for images and assets
- **Email**: AWS SES for notifications
- **Logging**: log4js with structured logging
- **Testing**: Jest with supertest for API testing

### Frontend Architecture
- **Next.js 14** with App Router and React 18
- **UI Framework**: Material-UI v5 with custom theming
- **State Management**: React Context API and hooks
- **Maps**: Mapbox GL JS and Google Maps integration
- **Authentication**: Auth0 React SDK
- **Mobile**: Capacitor for iOS/Android apps
- **Testing**: Jest with React Testing Library

### Key Architectural Patterns
- **Atomic Design**: Components organized as atoms/molecules/organisms/templates/pages
- **Environment Configuration**: Multi-environment support (localhost/dev/staging/app)
- **DMO Integration**: Destination Marketing Organization data import system
- **Trip Planning**: Complex trip builder with drag-and-drop functionality
- **Booking System**: Integration with Duffel API for accommodation booking

### Database Schema
- **Core Entities**: accounts, dmos, listings, trips, user_events, bookings
- **Reference Tables**: Prefixed with `ref_` (e.g., ref_categories)
- **Bridge Tables**: Many-to-many relationships prefixed with `bridge_`
- **Versioning**: Liquibase changesets with semantic version tags
- **Naming**: snake_case for tables/columns, plural table names

### Deployment Architecture
- **AWS Infrastructure**: CDK-managed EC2, RDS, S3, CloudFront
- **Multi-Environment**: Separate stacks for dev/staging/production
- **Security**: DMZ architecture with private subnets
- **SSL/TLS**: Certificate Manager with Route 53 validation
- **CI/CD**: Manual deployment via SSH tunnels and update scripts

## Key Integration Points

### Auth0 Configuration
- Tenant naming: `trippl-[local|dev|staging|app]`
- Custom post-login action creates users in database
- M2M application for API access
- Environment-specific callback URLs

### DMO Data Pipeline
1. Source data in `_sourceData/[dmo-name]/`
2. Transform scripts convert to standard format
3. Store in `_targetData/[dmo-name]/`
4. Import scripts load into database
5. Images stored in `_targetImages/` and S3

### Map Integration
- Mapbox for primary mapping functionality
- Google Places API for location search
- Custom markers for different listing types
- Geolocation-based search with 50km radius

### Development Environment Setup
- Requires .env files with Auth0 and database credentials
- PostgreSQL database via Docker Compose
- AWS profile configuration for S3/SES access
- Node.js 20.x with npm package management