# NGINX Load Balancer Demo

A demonstration project showing NGINX load balancing across multiple Node.js servers with static file serving capabilities.

## Features

- Load balancing across 3 Node.js servers
- Static file serving
- Multiple routing examples
- Docker containerization
- Health check endpoint

## Project Structure

```
.
├── nginx/
│   ├── nginx.conf        # NGINX configuration
│   └── mime.types       # MIME types configuration
├── server/
│   ├── Dockerfile       # Node.js server Dockerfile
│   ├── index.js         # Express server code
│   └── package.json     # Node.js dependencies
├── static/
│   ├── index.html       # Main page
│   ├── styles.css       # CSS styles
│   ├── fruits/          # Fruits page
│   └── vegetables/      # Vegetables page
├── docker-compose.yml   # Docker services configuration
└── README.md           # This file
```

## Available Routes

- `/` - Load balanced Node.js "Hello World" response
- `/health` - Server health check endpoint
- `/fruits` - Static fruits list page
- `/vegetables` - Static vegetables list page
- `/carbs` - Alias to fruits page
- `/crops` - Redirects to fruits page (307)
- `/number/{id}` - Rewritten to `/count/{id}`

## Prerequisites

- Docker
- Docker Compose

## Getting Started

1. Clone the repository:
```bash
git clone <repository-url>
cd <project-directory>
```

2. Start the services:
```bash
docker compose up -d
```

3. Access the application:
- Main application: http://nginx-test.local:8080
- Fruits page: http://nginx-test.local/fruits
- Vegetables page: http://nginx-test.local/vegetables

### NGINX Configuration

The NGINX configuration (`nginx/nginx.conf`) includes:
- Upstream server definitions
- Static file serving
- URL rewriting
- Location-based routing

## Load Balancing

The application demonstrates NGINX load balancing across three Node.js servers:
- Server 1: Port 1111 (maps to container port 3000)
- Server 2: Port 2222 (maps to container port 3000)
- Server 3: Port 3333 (maps to container port 3000)

Each request to the root path (`/`) is distributed across these servers.
You can see which server handled your request by checking the `X-Server-ID` response header.

Note: While each Node.js application runs on port 3000 inside its respective container, Docker maps these to different host ports (1111, 2222, 3333) to avoid conflicts. NGINX uses these host ports to communicate with the containers.

### Static Files

Static files are served from the `static/` directory. Add or modify files here to update the static content.

## Stopping the Application

```bash
docker compose down
```