# Nginx Load Balancer Demo

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

2. Configure local hosts file:
   - On Unix/Linux/Mac: Edit `/etc/hosts`
   - On Windows: Edit `C:\Windows\System32\drivers\etc\hosts`

   Add the following entries:
   ```
   127.0.0.1    nginx-test.local
   127.0.0.1    api.nginx-test.local
   127.0.0.1    api-https.nginx-test.local
   127.0.0.1    api-https.nginx-test-ip-spoofing.local
   ```

   Note: You might need administrator/sudo privileges to edit the hosts file.

3. Start the services:
```bash
docker compose up -d
```

4. Access the application:
- Main application: http://nginx-test.local
- Fruits page: http://nginx-test.local/fruits
- Vegetables page: http://nginx-test.local/vegetables

### Testing Endpoints

You can test the endpoints using the provided `requests.http` file:
- HTTP endpoints: `http://nginx-test.local/health`
- HTTPS endpoints: `https://api-https.nginx-test.local/health`

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

Note: When switching between HTTP and HTTPS configurations, make sure to update your request URLs accordingly.

## Stopping the Application

```bash
docker compose down
```

## SSL/HTTPS Setup

### Generating Self-Signed Certificates (Development Only)
For local development with HTTPS, you'll need to generate self-signed SSL certificates:

```bash
# Create ssl directory if it doesn't exist
mkdir -p ssl

# Generate self-signed certificate and private key
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/nginx-selfsigned.key \
  -out ssl/nginx-selfsigned.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=nginx-test.local"
```

Note: Self-signed certificates will trigger browser warnings. This is expected in development. For production, use certificates from a trusted provider like Let's Encrypt.

### Using HTTPS Locally
1. Generate the SSL certificates as shown above
2. Ensure the SSL configuration in `nginx.conf` points to the correct certificate files:
```nginx
ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
```
3. Make sure port 443 is exposed in `docker-compose.yml`
4. Restart the containers:
```bash
docker compose down
docker compose up -d
```

You can then test HTTPS endpoints:
```http
https://api-https.nginx-test.local/health
```

Note: When using self-signed certificates, you may need to:
- Accept the security warning in your browser
- Use the `-k` flag with curl commands
- Configure your HTTP client to accept invalid certificates

## IP Spoofing Protection

### What is IP Spoofing?
IP spoofing is a cyber attack where malicious actors forge their IP address to:
- Impersonate trusted sources
- Bypass IP-based security controls
- Hide their true identity
- Potentially conduct DDoS attacks

For example, without protection, attackers could forge requests that appear to come from Vercel's infrastructure, potentially bypassing security measures.

### Vercel Proxy Configuration
The application includes protection against IP spoofing when using Vercel as a proxy. This is configured at:
```http
http://api-https.nginx-test-ip-spoofing.local
```

### Security Features
1. **IP Restriction**
   ```nginx
   # Only allows traffic from Vercel's infrastructure
   allow 76.76.21.0/24;      # Vercel IP range 1
   allow 76.76.22.0/24;      # Vercel IP range 2
   deny all;                 # Block all other traffic
   ```

2. **Header Protection**
   ```nginx
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header Host $host;
   ```

### Testing IP Protection
1. Add the domain to your hosts file:
   ```
   127.0.0.1    api-https.nginx-test-ip-spoofing.local
   ```

2. Requests will only be allowed if they originate from Vercel's IP ranges:
   - 76.76.21.0/24
   - 76.76.22.0/24

### Security Notes
- Keep Vercel IP ranges updated as they may change
- Monitor access logs for potential security issues
- Consider implementing rate limiting for additional protection
- For production, combine with SSL/HTTPS configuration