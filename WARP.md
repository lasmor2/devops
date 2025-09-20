# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Commands

### Core Development
- **Start development server**: `npm run dev` - Runs with file watching using Node.js --watch flag
- **Lint code**: `npm run lint` - Runs ESLint for code quality checks
- **Fix linting issues**: `npm run lint:fix` - Automatically fixes ESLint issues
- **Format code**: `npm run format` - Formats code with Prettier
- **Check formatting**: `npm run format:check` - Validates code formatting without changes

### Database Operations (Drizzle ORM)
- **Generate migrations**: `npm run db:generate` - Creates new migration files from schema changes
- **Apply migrations**: `npm run db:migrate` - Applies pending migrations to database
- **Database Studio**: `npm run db:studio` - Opens Drizzle Studio for database exploration

## Architecture Overview

This is a Node.js Express.js REST API with a PostgreSQL database using Drizzle ORM. The application follows a modular architecture with clear separation of concerns.

### Core Technologies
- **Runtime**: Node.js with ES modules
- **Framework**: Express.js 5.x
- **Database**: PostgreSQL via Neon serverless
- **ORM**: Drizzle ORM with drizzle-kit
- **Validation**: Zod schemas
- **Authentication**: JWT tokens with bcrypt password hashing
- **Logging**: Winston logger
- **Security**: Helmet, CORS, cookie-parser

### Project Structure & Import Aliases

The project uses Node.js subpath imports for clean module resolution:

```javascript
#src/*          → ./src/*
#config/*       → ./src/config/*
#controllers/*  → ./src/controllers/*
#middleware/*   → ./src/middleware/*
#models/*       → ./src/models/*
#routes/*       → ./src/routes/*
#services/*     → ./src/services/*
#utils/*        → ./src/utils/*
#validations/*  → ./src/validations/*
```

### Application Flow
1. **Entry Point**: `src/index.js` → loads environment variables → starts `src/server.js`
2. **Server Setup**: `src/server.js` → initializes Express app from `src/app.js`
3. **App Configuration**: `src/app.js` → middleware setup, security, routing

### Database Architecture
- **Connection**: Neon serverless PostgreSQL via `@neondatabase/serverless`
- **ORM Configuration**: `drizzle.config.js` points to `src/models/*.js` for schema
- **Migration Storage**: `./drizzle/` directory contains generated SQL migrations
- **Current Schema**: Single `users` table with authentication fields

### Authentication System
- **JWT-based**: Tokens stored in HTTP-only cookies with 15-minute expiration
- **Password Security**: bcrypt with 10 rounds for hashing
- **User Roles**: Enum-based role system (user/admin) with default 'user'
- **Cookie Security**: HttpOnly, SameSite strict, secure in production

### Validation Strategy
- **Schema Validation**: Zod schemas in `src/validations/` for all endpoints
- **Error Formatting**: Centralized validation error formatting in `src/utils/format.js`
- **Input Sanitization**: Email normalization, string trimming, length constraints

### Logging Configuration
- **Winston Logger**: JSON format with timestamps and error stack traces
- **Log Levels**: Configurable via `LOG_LEVEL` environment variable (default: info)
- **File Outputs**: `logs/error.log` for errors, `logs/combined.log` for all logs
- **Development**: Console output with colorized, simple format

### Security Middleware Stack
1. **Helmet**: Security headers
2. **CORS**: Cross-origin resource sharing
3. **Express.json()**: Request body parsing with built-in limits
4. **Cookie Parser**: Secure cookie handling
5. **Morgan**: HTTP request logging integrated with Winston

### Environment Variables Required
```env
DATABASE_URL=postgresql://...    # Neon database connection
JWT_SECRET=your_secret_here      # JWT signing secret
LOG_LEVEL=info                   # Optional: logging level
NODE_ENV=production              # Optional: environment mode
PORT=4000                        # Optional: server port
```

### Code Style & Standards
- **ESLint**: Enforces 2-space indentation, single quotes, semicolons
- **Prettier**: Consistent code formatting
- **Line Endings**: Windows-style (CRLF) enforced via ESLint
- **ES Modules**: Strict ES module usage with import/export syntax
- **Arrow Functions**: Preferred over function declarations where applicable

### API Endpoints Structure
- **Health Check**: `GET /health` - Application status and uptime
- **API Base**: `GET /api` - API status confirmation  
- **Authentication**: `POST /api/auth/sign-up`, `POST /api/auth/sign-in`, `POST /api/auth/sign-out`

### Adding New Features

When adding new functionality:

1. **Database Changes**: Update models in `src/models/`, run `npm run db:generate`, then `npm run db:migrate`
2. **API Endpoints**: Create validation schema → service layer → controller → route registration
3. **Follow Patterns**: Mirror the auth system's structure for consistency
4. **Error Handling**: Use consistent error formatting and logging patterns
5. **Testing**: ESLint configuration includes Jest globals for future test implementation

### Development Workflow
1. Make code changes using the existing patterns
2. Run `npm run lint:fix` and `npm run format` before committing
3. Test database changes with `npm run db:studio`
4. Use `npm run dev` for development with automatic file watching