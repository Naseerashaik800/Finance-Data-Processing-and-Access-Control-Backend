# Finance Backend - Quick Start Guide

## Prerequisites

Before you start, ensure you have:
- **Node.js** (v16 or higher) - Download from [nodejs.org](https://nodejs.org/)
- **PostgreSQL** (v12 or higher) - Download from [postgresql.org](https://www.postgresql.org/)
- **npm** (comes with Node.js)

## Option 1: Run PostgreSQL Locally

### macOS (with Homebrew)
```bash
# Install PostgreSQL
brew install postgresql@15

# Start the service
brew services start postgresql@15

# Create the database
createdb finance_db

# Verify connection
psql -U postgres -d finance_db -c "SELECT version();"
```

### Windows
```bash
# PostgreSQL installer will add psql to PATH
# After installation, open Command Prompt or PowerShell

# Create the database
createdb -U postgres finance_db

# Verify connection
psql -U postgres -d finance_db -c "SELECT version();"
```

### Linux (Ubuntu)
```bash
# Install PostgreSQL
sudo apt-get install postgresql postgresql-contrib

# Start the service
sudo systemctl start postgresql

# Create the database
sudo -u postgres createdb finance_db

# Verify connection
sudo -u postgres psql -d finance_db -c "SELECT version();"
```

## Option 2: Run PostgreSQL with Docker

If you have Docker installed, this is the easiest option:

```bash
# Create and run PostgreSQL container
docker run --name finance-postgres \
  -e POSTGRES_DB=finance_db \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  -d postgres:15-alpine

# Verify the container is running
docker ps

# Stop the container (when done)
docker stop finance-postgres

# Start it again (it persists data)
docker start finance-postgres

# Remove the container completely (with -v to remove volume)
docker rm -v finance-postgres
```

## Step-by-Step Setup

### 1. Clone or Download the Project

```bash
# If cloning from git
git clone <repository-url>
cd finance-backend

# Or navigate to your project folder
cd "c:\Users\bharg\OneDrive\React js\Project2"
```

### 2. Install Dependencies

```bash
npm install
```

This will install all required packages (Express, TypeScript, PostgreSQL driver, etc.).

### 3. Configure Environment

The project includes a `.env` file with development defaults. Verify the values match your PostgreSQL setup:

```bash
# Open .env and verify these settings:
DB_HOST=localhost              # Your PostgreSQL host
DB_PORT=5432                   # PostgreSQL port (default: 5432)
DB_NAME=finance_db             # Database name you created
DB_USER=postgres               # PostgreSQL user (default: postgres)
DB_PASSWORD=postgres           # PostgreSQL password
PORT=3000                      # Express server port
NODE_ENV=development           # Development mode
JWT_SECRET=dev_jwt_secret_key  # Change this in production!
JWT_EXPIRATION=7d              # Token expiration time
```

**Important**: If you set a different PostgreSQL password, update `DB_PASSWORD` in `.env`.

### 4. Start the Development Server

```bash
npm run dev
```

You should see output like:
```
✓ Database schema initialized successfully!
✓ Server running on http://localhost:3000
✓ Environment: development
```

**Success!** The server is now running. 🎉

### 5. Test the API

In a new terminal, test with cURL or Postman:

```bash
# Health check
curl http://localhost:3000/health

# Register a new user
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "password123"
  }'

# Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "password123"
  }'
```

## Available Commands

```bash
# Development
npm run dev          # Start dev server with auto-reload
npm run typecheck    # Check TypeScript types
npm run lint         # Run ESLint

# Production
npm run build        # Compile TypeScript to JavaScript
npm start            # Run production build

# Helpers
npm install          # Install dependencies
npm audit            # Check for security vulnerabilities
```

## Default Test Credentials

If you want to seed sample data, uncomment the seedDatabase call in [src/index.ts](src/index.ts#L43):

```typescript
// await seedDatabase();  // Uncomment to seed sample data
```

Then restart the server. Default test accounts:
- **Admin**: username: `admin_user`, password: `admin123`
- **Analyst**: username: `analyst_user`, password: `analyst123`
- **Viewer**: username: `viewer_user`, password: `viewer123`

## Project Structure

```
finance-backend/
├── src/
│   ├── database/         # Database connection and schemas
│   ├── middleware/       # Authentication, validation, error handling
│   ├── routes/           # API endpoint definitions
│   ├── services/         # Business logic
│   ├── types/            # TypeScript type definitions
│   ├── utils/            # Utility functions
│   └── index.ts          # Application entry point
├── dist/                 # Compiled JavaScript (generated by npm run build)
├── .env                  # Environment variables (local)
├── .env.example          # Environment template
├── package.json          # Project dependencies
├── tsconfig.json         # TypeScript configuration
├── .eslintrc.json        # ESLint configuration
├── README.md             # Full documentation
├── API_DOCUMENTATION.md  # API reference
└── this file
```

## Troubleshooting

### "Can't connect to PostgreSQL"
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```
**Solution**: 
- Ensure PostgreSQL is running (`brew services list` on macOS, `systemctl status postgresql` on Linux)
- Check DB_HOST, DB_PORT, DB_USER, DB_PASSWORD in `.env`
- On Windows, PostgreSQL may run as a service - check Services app

### "Database finance_db does not exist"
**Solution**:
```bash
# Create the database
createdb finance_db  # (or: createdb -U postgres finance_db)
```

### "Port 3000 is already in use"
**Solution**:
```bash
# Change PORT in .env, or kill the process using port 3000:
# On macOS/Linux:
lsof -ti:3000 | xargs kill -9

# On Windows (PowerShell):
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

### "npm ERR! Cannot find module"
**Solution**:
```bash
# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

### TypeScript compilation errors
**Solution**:
```bash
# Check for errors
npm run typecheck

# Rebuild
npm run build
```

## Next Steps

1. **Read the full documentation**: [README.md](README.md)
2. **Explore the API**: [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
3. **Import endpoints into Postman**: See Postman examples in API documentation
4. **For production deployment**: See production checklist in README.md

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   CLIENT (Frontend)                          │
└────────────┬────────────────────────────────────────────────┘
             │ HTTP/REST
┌────────────▼────────────────────────────────────────────────┐
│              Express.js Server (Port 3000)                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Routes: /api/auth, /api/users, /api/records,       │   │
│  │         /api/dashboard                              │   │
│  └──────────────────┬──────────────────────────────────┘   │
│                     │                                        │
│  ┌──────────────────▼──────────────────────────────────┐   │
│  │ Middleware: Auth, Validation, Error Handling       │   │
│  └──────────────────┬──────────────────────────────────┘   │
│                     │                                        │
│  ┌──────────────────▼──────────────────────────────────┐   │
│  │ Services: User, Record, Dashboard Logic             │   │
│  └──────────────────┬──────────────────────────────────┘   │
│                     │                                        │
│  ┌──────────────────▼──────────────────────────────────┐   │
│  │ Database Connection & Queries                        │   │
│  └──────────────────┬──────────────────────────────────┘   │
└────────────┬────────────────────────────────────────────────┘
             │ SQL
┌────────────▼────────────────────────────────────────────────┐
│              PostgreSQL Database                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Tables: users, financial_records                    │   │
│  │ Indexes on: user_id, date, category, type          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Support

- **Documentation**: See README.md and API_DOCUMENTATION.md
- **Code Structure**: Follow TypeScript + Express best practices
- **Type Safety**: All code is fully typed with TypeScript
- **Error Handling**: Consistent error responses with meaningful messages

Happy coding! 🚀
