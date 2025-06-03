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
   The configuration uses separate upstream blocks for each service:

   ```nginx
   # Define upstream servers
   upstream auth_backend {
       server host.docker.internal:5001;
       server 172.17.0.1:5001 backup;
   }

   upstream jobs_backend {
       server host.docker.internal:5002;
       server 172.17.0.1:5002 backup;
   }

   upstream cv_backend {
       server host.docker.internal:5003;
       server 172.17.0.1:5003 backup;
   }
   ```

   The configuration includes:
   - Separate upstream blocks for each service
   - CORS support for localhost:3000
   - Common proxy settings and timeouts
   - Automatic handling of OPTIONS requests
   - Backup server support for Linux systems

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
       if ($request_method = 'OPTIONS') {
           add_header 'Access-Control-Allow-Origin' $cors_origin;
           add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD';
           add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization,Accept';
           add_header 'Access-Control-Allow-Credentials' 'true';
           add_header 'Access-Control-Max-Age' 1728000;
           add_header 'Content-Type' 'text/plain; charset=utf-8';
           add_header 'Content-Length' 0;
           return 204;
       }

       proxy_pass http://new_service_backend/new-service/;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_set_header X-Forwarded-Host $host;
       proxy_set_header X-Forwarded-Port $server_port;
   }
   ```

3. Ensure your service is running on the specified port (e.g., 5004)

4. Restart the nginx container:
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
