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

The reverse proxy is configured to run on port 5000 and route traffic to multiple backend services.

1. Docker Compose Configuration (`docker-compose.yml`):
   ```yaml
   version: '3.8'
   
   services:
     nginx:
       image: nginx:latest
       container_name: nginx_proxy
       ports:
         - "5000:5000"
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
   ```nginx
   events {}

   http {
       upstream auth_backend {
           server host.docker.internal:5001;
           server 172.17.0.1:5001 backup;
       }

       server {
           listen 5000;
           server_name localhost;

           location /auth/ {
               proxy_pass         http://auth_backend/auth/;
               proxy_set_header   Host $host;
               proxy_set_header   X-Real-IP $remote_addr;
               proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header   X-Forwarded-Proto $scheme;
           }
       }
   }
   ```

### Adding New Services

To add a new service:

1. Add a new upstream block in `nginx.conf`:
   ```nginx
   upstream new_service_backend {
       server host.docker.internal:5004;    # Primary connection for Windows/macOS
       server 172.17.0.1:5004 backup;       # Backup connection for Linux
   }
   ```

2. Add a new location block in the server section:
   ```nginx
   location /new-service/ {
       proxy_pass         http://new_service_backend/new-service/;
       proxy_set_header   Host $host;
       proxy_set_header   X-Real-IP $remote_addr;
       proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header   X-Forwarded-Proto $scheme;
   }
   ```

3. Ensure your service is running on the specified port (e.g., 5004)

4. Restart the nginx container using the commands in the Getting Started section

Note: The current setup uses `host.docker.internal` for Windows/macOS and falls back to `172.17.0.1` for Linux systems.

## License

[MIT License](LICENSE)
