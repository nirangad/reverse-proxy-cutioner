# Reverse Proxy-Cutioner

A tool to manage and orchestrate reverse proxy configurations using Docker containers.

## Prerequisites

- Docker
- Docker Compose

## Getting Started

### Running the Application

To start the application, run:

```bash
docker-compose -p reverse-proxy-cutioner up --build -d
```

This command will:
- Build the necessary Docker images
- Create and start the containers in detached mode
- Use the project name "reverse-proxy-cutioner"

### Stopping the Application

To stop and clean up all resources, run:

```bash
docker-compose -p reverse-proxy-cutioner down -v --rmi all --remove-orphans
```

This command will:
- Stop all containers
- Remove all containers, networks, and volumes
- Remove all images
- Remove any orphaned containers

## Configuration

### Current Setup

The reverse proxy is configured to run on port 4567 and route traffic to multiple backend services.

1. Docker Compose Configuration (`docker-compose.yml`):
   ```yaml
   services:
     nginx:
       image: nginx:1.27.5
       container_name: nginx_proxy
       ports:
         - "4567:5000"
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf:ro
       networks:
         - app-network
       restart: unless-stopped

   networks:
     app-network:
       driver: bridge
   ```

2. Nginx Configuration (`nginx.conf`):
   The configuration uses a dynamic approach to handle multiple services:

   ```nginx
   # Service to port mapping
   map $service $backend_port {
       auth 5001;
       jobs 5002;
       cv 5003;
       # Add new services here
   }

   # Common upstream configuration
   upstream backend {
       server host.docker.internal:$backend_port;
       server 172.17.0.1:$backend_port backup;
   }
   ```

   The configuration includes:
   - Dynamic service routing based on URL paths
   - CORS support for localhost:3000
   - Common proxy settings and timeouts
   - Automatic handling of OPTIONS requests
   - Backup server support for Linux systems

### Adding New Services

To add a new service:

1. Add a new service mapping in the `map` block in `nginx.conf`:
   ```nginx
   map $service $backend_port {
       # ... existing services ...
       new_service 5004;    # Add your new service here
   }
   ```

2. Ensure your service is running on the specified port (e.g., 5004)

3. Restart the nginx container:
   ```bash
   docker-compose -p reverse-proxy-cutioner restart nginx
   ```

The configuration will automatically:
- Route requests to the correct backend based on the URL path
- Handle CORS headers
- Manage proxy settings
- Support both Windows/macOS (`host.docker.internal`) and Linux (`172.17.0.1`) environments

## License

[MIT License](LICENSE)
