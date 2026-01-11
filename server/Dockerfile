# ============================================
# Dockerfile for Hello World Node.js Server
# ============================================
# This Dockerfile demonstrates how to containerize a Node.js application
# Each instruction creates a new layer in the Docker image

# Step 1: Choose the base image
# We use the official Node.js runtime as our base image
# '18-alpine' means Node.js version 18 on Alpine Linux (lightweight)
# Alpine is smaller (~5MB) compared to full Debian-based images (~100MB+)
FROM node:18-alpine

# Step 2: Set the working directory inside the container
# All subsequent commands will run in this directory
# If the directory doesn't exist, Docker will create it
WORKDIR /app

# ============================================
# Environment Variables in Dockerfile
# ============================================
# ARG: Build-time variables (only available during build)
# ENV: Runtime variables (available in container at runtime)

# Set default environment variables
# These can be overridden at runtime using -e flag: docker run -e NODE_ENV=development
# ENV variables are available to the application inside the container
ENV NODE_ENV=production
ENV DATABASE_URL=""
ENV PORT=3000


# Note: DATABASE_URL is set to empty string as default
# It should be provided at runtime: docker run -e DATABASE_URL=postgresql://...
# You can also override NODE_ENV: docker run -e NODE_ENV=development

# Step 3: Copy package files first (for better caching)
# Docker caches layers, so if package.json doesn't change,
# Docker will reuse the cached layer and skip reinstalling dependencies
# This is called "layer caching" - a Docker best practice
COPY package*.json ./

# Step 4: Install dependencies
# This installs all packages listed in package.json
# The --production flag would skip devDependencies (not needed here)
RUN npm install

# Step 5: Copy the rest of the application code
# We copy package.json first (above) and source code now
# This way, if only source code changes, we don't reinstall dependencies
COPY . .

# Step 6: Expose the port the app runs on
# This doesn't actually publish the port, it's just documentation
# The port will be published when running the container with -p flag
# Example: docker run -p 3000:3000 hello-world-server
EXPOSE 3000

# Step 7: Define the command to run when container starts
# This is the default command if no command is specified at runtime
# Using the npm start script defined in package.json
CMD ["npm", "start"]

# ============================================
# Building and Running the Container
# ============================================
#
# Build the image:
#   docker build -t hello-world-server .
#
# Run with default settings:
#   docker run -p 3000:3000 hello-world-server
#
# Run with inline environment variables:
#   docker run -p 3000:3000 -e PORT=8080 -e NODE_ENV=development hello-world-server
#
# Run with required environment variables (NODE_ENV and DATABASE_URL):
#   docker run -p 3000:3000 \
#     -e NODE_ENV=production \
#     -e DATABASE_URL=postgresql://user:pass@localhost:5432/mydb \
#     hello-world-server
#
# Run with .env file:
#   docker run -p 3000:3000 --env-file .env hello-world-server
#
# Run in detached mode (background):
#   docker run -d -p 3000:3000 --env-file .env hello-world-server
#
# Access the app:
#   http://localhost:3000
#
# ============================================
# Environment Variable Methods:
# ============================================
# 1. Inline: -e KEY=value (e.g., -e NODE_ENV=production)
# 2. .env file: --env-file .env
# 3. Shell variables: -e PORT=$PORT
# 4. Multiple variables: -e VAR1=val1 -e VAR2=val2
# ============================================
