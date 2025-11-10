# Simple Zabbix Deployment with TimescaleDB

This is a simplified, single-file Docker Compose deployment for Zabbix 7.4 (Ubuntu) with PostgreSQL TimescaleDB support.

## Components

- **PostgreSQL with TimescaleDB**: `timescale/timescaledb:2.22.0-pg17`
- **Zabbix Server**: `zabbix/zabbix-server-pgsql:ubuntu-7.4-latest`
- **Zabbix Frontend**: `zabbix/zabbix-web-nginx-pgsql:ubuntu-7.4-latest`
- **Zabbix Agent**: `zabbix/zabbix-agent:ubuntu-7.4-latest`

## Quick Start

### Prerequisites

- Docker Engine 20.10 or later
- Docker Compose v2.0 or later

### Deployment

1. Start the stack:
   ```bash
   docker compose -f docker-compose-simple-timescaledb.yaml up -d
   ```

2. Wait for all services to start (this may take 1-2 minutes on first run as the database needs to initialize)

3. Access the Zabbix web interface:
   - URL: http://localhost
   - Default credentials:
     - Username: `Admin`
     - Password: `zabbix`

### Check Status

```bash
# View logs
docker compose -f docker-compose-simple-timescaledb.yaml logs -f

# Check service status
docker compose -f docker-compose-simple-timescaledb.yaml ps
```

### Stop the Stack

```bash
docker compose -f docker-compose-simple-timescaledb.yaml down
```

### Stop and Remove All Data

```bash
docker compose -f docker-compose-simple-timescaledb.yaml down -v
```

## Configuration

### Database Credentials

Default credentials (can be changed in the compose file):
- Database: `zabbix`
- Username: `zabbix`
- Password: `zabbix_password`

### Ports

- **80**: Zabbix web interface (HTTP)
- **443**: Zabbix web interface (HTTPS)
- **10050**: Zabbix agent (for monitoring the host)
- **10051**: Zabbix server trapper port

### TimescaleDB

TimescaleDB is enabled by default with the `ENABLE_TIMESCALEDB: "true"` environment variable. This provides optimized time-series data storage for Zabbix metrics.

### Zabbix Agent

The deployment includes a Zabbix agent configured to monitor the host running the Docker containers. The agent is configured with:
- Hostname: "Zabbix server"
- Server: zabbix-server
- Port: 10050

The agent runs in privileged mode with host PID namespace to access host metrics. By default, Zabbix creates a host named "Zabbix server" that monitors itself. This agent provides the actual monitoring data for that host.

**Important**: After initial deployment, you need to update the default "Zabbix server" host interface in the Zabbix web UI:
1. Log in to the web interface (Admin/zabbix)
2. Go to **Data collection** → **Hosts**
3. Click on **Zabbix server** host
4. Go to the **Interfaces** tab
5. Change the IP address from `127.0.0.1` to `zabbix-agent` (the container name)
6. Click **Update**

This ensures the Zabbix server can properly connect to the agent container for monitoring.

### SNMP MIBs

Custom SNMP MIBs can be placed in the `zabbix-mibs` volume. To add your MIBs:

```bash
# Copy MIB files to the volume
docker cp your-mib-file.txt $(docker compose -f docker-compose-simple-timescaledb.yaml ps -q zabbix-server):/var/lib/zabbix/mibs/
```

Or use a bind mount by editing the compose file to replace the named volume with a local directory:
```yaml
volumes:
  - ./mibs:/var/lib/zabbix/mibs:ro
```

### SSL Certificates

SSL certificates for HTTPS can be placed in the `zabbix-web-ssl` volume at `/etc/ssl/nginx`. To use custom certificates:

```bash
# Copy certificate files to the volume
docker cp your-cert.crt $(docker compose -f docker-compose-simple-timescaledb.yaml ps -q zabbix-web):/etc/ssl/nginx/
docker cp your-key.key $(docker compose -f docker-compose-simple-timescaledb.yaml ps -q zabbix-web):/etc/ssl/nginx/
```

Or use a bind mount for easier management:
```yaml
volumes:
  - ./ssl:/etc/ssl/nginx
```

## Customization

Edit the `docker-compose-simple-timescaledb.yaml` file to customize:

- Database credentials
- Timezone (PHP_TZ)
- Server name
- Port mappings
- Resource limits

## Troubleshooting

### Container won't start

Check logs:
```bash
docker compose -f docker-compose-simple-timescaledb.yaml logs
```

### Database connection issues

Ensure the postgres-server container is running and healthy:
```bash
docker compose -f docker-compose-simple-timescaledb.yaml ps postgres-server
```

### Web interface not accessible

Check if the zabbix-web container is running:
```bash
docker compose -f docker-compose-simple-timescaledb.yaml ps zabbix-web
```

View web container logs:
```bash
docker compose -f docker-compose-simple-timescaledb.yaml logs zabbix-web
```

## Data Persistence

Data is persisted in Docker volumes:
- `postgres-data`: PostgreSQL database files
- `zabbix-server-data`: Zabbix server configuration and data
- `zabbix-mibs`: SNMP MIBs directory (place your custom MIBs here)
- `zabbix-web-ssl`: SSL certificates for the web interface

To backup your data, backup these volumes or the data stored in them.

## Security Notes

**Important**: Change the default database password before deploying to production!

1. Edit the compose file and change `POSTGRES_PASSWORD` in both the postgres-server and all Zabbix services
2. Change the default Zabbix Admin password immediately after first login
3. Consider using Docker secrets for production deployments
4. Configure HTTPS with proper SSL certificates
5. Restrict access using firewalls or network policies

## Additional Components

This is a minimal deployment. For a full-featured setup, see the other compose files in this repository that include:
- Zabbix Proxy
- Zabbix Agent
- Java Gateway
- SNMP Traps support
- Web Service for reporting

## Support

For issues related to:
- Docker images: https://github.com/zabbix/zabbix-docker/issues
- Zabbix itself: https://support.zabbix.com/
