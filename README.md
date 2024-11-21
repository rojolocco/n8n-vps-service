# n8n Self-Hosted App with Docker Compose and Caddy

This repository contains the configuration files to run an instance of the **n8n** workflow automation tool on a VPS using Docker Compose, with **Caddy** as a reverse proxy to manage HTTPS.

## Prerequisites

- Docker and Docker Compose installed.
- Caddy installed on the VPS.
- A subdomain (e.g., `n8n.mivps.com`) configured to point to your VPS IP address.

## Setup Instructions

### 1. Clone the Repository

Start by cloning the repository to your VPS:

```sh
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

### 2. Create the Docker Volume

Create a Docker volume to persist n8n data:

```sh
docker volume create n8n_data
```

### 3. Update the Docker Compose File

Ensure you have the `docker-compose.yml` file configured as follows:

```yaml
version: "3"

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=America/Bogota  # Update according to your timezone
      - TZ=America/Bogota
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

This configuration ensures that your workflows and settings are persisted in the `n8n_data` volume.

### 4. Configure Caddy for Reverse Proxy

Update your Caddyfile (usually located in `/etc/caddy/Caddyfile`) to proxy traffic to n8n:

```caddy
n8n.mivps.com {
    reverse_proxy localhost:5678
}
```

After saving the changes, restart Caddy to apply the configuration:

```sh
sudo systemctl restart caddy
```

### 5. Launch n8n

To launch n8n, run:

```sh
docker compose up -d
```

This command will start the n8n service in detached mode, making it accessible at `https://n8n.mivps.com`.

### 6. Update n8n

To update to the latest version of n8n, use the following commands:

```sh
# Pull the latest version
docker compose pull

# Stop and remove the older version
docker compose down

# Start the updated version
docker compose up -d
```

### 7. Ensure Users Must Log In Every Session

To ensure users must log in every time after the container restarts without losing workflow data, you can clear only the session data. One way to achieve this is by using environment variables to control session management or by scripting a solution to delete session-specific files without affecting workflows. Alternatively, you can set a shorter session expiration policy in n8n to reduce the time users stay logged in.

## Troubleshooting

- **Cannot access n8n**: Ensure the Docker container is running using `docker ps`. Check that Caddy is correctly configured and running.
- **Session not cleared**: Consider implementing custom session management by controlling session storage or setting a shorter session expiration time.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

