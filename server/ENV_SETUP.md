# Environment Variables Setup

## Required Environment Variables

This application requires the following environment variables:

### NODE_ENV
- **Description**: Node.js environment mode
- **Options**: `production`, `development`, `test`
- **Default**: `production`
- **Example**: `NODE_ENV=production`

### DATABASE_URL
- **Description**: Database connection string
- **Required**: Yes (no default value)
- **Format**: `protocol://user:password@host:port/database`

#### Examples:
- **PostgreSQL**: `postgresql://user:password@localhost:5432/mydb`
- **MySQL**: `mysql://user:password@localhost:3306/mydb`
- **MongoDB**: `mongodb://user:password@localhost:27017/mydb`

### PORT (Optional)
- **Description**: Server port number
- **Default**: `3000`
- **Example**: `PORT=3000`

## Setup Instructions

1. Create a `.env` file in the `server` directory:
   ```bash
   cd server
   touch .env
   ```

2. Add the following content to `.env`:
   ```env
   # Server Configuration
   PORT=3000

   # Node Environment
   NODE_ENV=production

   # Database Configuration
   DATABASE_URL=postgresql://user:password@localhost:5432/mydb
   ```

3. Update the values with your actual configuration

4. The `.env` file is already in `.gitignore`, so it won't be committed to version control

## Usage with Docker Compose

Docker Compose automatically loads the `.env` file. The `env_file` directive in `docker-compose.yml` explicitly loads it:

```yaml
env_file:
  - .env
```

Environment variables can also be set inline in `docker-compose.yml` using the `${VARIABLE:-default}` syntax.
