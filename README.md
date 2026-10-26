# grafana-loki-monitoring
Centralized logging with Grafana, Loki, Grafana Alloy and Docker Compose

# Centralized Logging with Grafana, Loki and Alloy

A lightweight centralized logging solution using Docker Compose, Grafana, Loki, and Grafana Alloy.

The project collects Linux system logs and Docker container logs and makes them searchable through Grafana.

## Architecture

```text
Ubuntu Linux Host
│
├── System Logs
│   ├── /var/log/syslog
│   ├── /var/log/auth.log
│   └── /var/log/kern.log
│
├── Docker Containers
│   ├── Odoo
│   └── PostgreSQL
│
└── Grafana Alloy
        │
        │ Push logs
        ▼
      Grafana Loki
        │
        │ LogQL queries
        ▼
      Grafana
        │
        └── Explore and Dashboards
```

## Components

| Component | Purpose |
|---|---|
| Grafana | Log visualization, dashboards, and exploration |
| Loki | Centralized log storage and querying |
| Grafana Alloy | Collection and forwarding of logs |
| Docker Compose | Service deployment and lifecycle management |

## Environment

The initial deployment uses an Ubuntu Linux server and Docker Compose.

Example service endpoints:

| Service | Endpoint |
|---|---|
| Grafana | `http://SERVER_IP:3000` |
| Loki | `http://SERVER_IP:3100` |
| Alloy UI | `http://127.0.0.1:12345` |

Replace `SERVER_IP` with the actual server address.

The Alloy UI is bound to localhost in the example deployment and is not intended to be exposed publicly.

## Features

- Centralized collection of Linux system logs
- Docker container log collection
- Log search through Grafana Explore
- LogQL queries for troubleshooting
- Dashboard support through Grafana JSON imports
- Docker Compose-based deployment
- Configurable log retention

## Prerequisites

- Ubuntu Linux
- Docker Engine
- Docker Compose plugin
- Sufficient disk space for log retention
- Network connectivity between Alloy, Loki, and Grafana

## Deployment

1. Install Docker Engine and the Docker Compose plugin.
2. Create the project directories.
3. Add the Loki configuration file.
4. Add the Alloy configuration file.
5. Configure the Docker Compose services.
6. Start the services.
7. Configure Loki as a Grafana data source.

Start the stack from the directory containing the Compose file:

```bash
docker compose config
docker compose up -d
docker compose ps
```

Check the service logs:

```bash
docker compose logs --tail=100
```

If the services are defined in separate Compose projects, run these commands from each project's directory.

## Configure Loki in Grafana

1. Open Grafana.
2. Navigate to Connections → Data sources.
3. Add a Loki data source.
4. Set the URL to the Loki endpoint reachable from Grafana.
5. Save and test the connection.

If Grafana and Loki run in separate containers, use an address reachable from the Grafana container. A Docker network service name may be preferable to a host IP when both services share a Docker network.

## Example LogQL Queries

### Linux system logs

```logql
{job="system", host="odooo"}
```

### Authentication logs

```logql
{job="system", log_type="auth", host="odooo"}
```

### Kernel logs

```logql
{job="system", log_type="kernel", host="odooo"}
```

### Docker container logs

```logql
{job="docker", host="odooo"}
```

### Odoo logs

```logql
{job="docker", container=~".*odoo18.*"}
```

### PostgreSQL logs

```logql
{job="docker", container=~".*odoo18-db.*"}
```

### Search for errors

```logql
{job="docker"} |~ "(?i)error|exception|failed|fatal"
```

### Log volume over time

```logql
sum(count_over_time({host="odooo"}[5m]))
```

The queries depend on the labels emitted by the active Alloy configuration. Adjust the selectors if your labels differ.

## Grafana Dashboards

Dashboard JSON files can be stored under:

```text
grafana/dashboards/
```

To import a dashboard:

1. Open Grafana.
2. Navigate to Dashboards → New → Import.
3. Upload the dashboard JSON or paste its contents.
4. Select the configured Loki data source.
5. Import the dashboard.
6. Select an appropriate time range, such as Last 1 hour.

## Troubleshooting

### Check containers

```bash
docker ps
```

### Check Alloy logs

```bash
docker logs --tail=100 alloy
```

### Check Loki logs

```bash
docker logs --tail=100 loki
```

### Check Grafana logs

```bash
docker logs --tail=100 grafana
```

### Check Loki readiness

```bash
curl http://127.0.0.1:3100/ready
```

### Check Alloy readiness

```bash
curl http://127.0.0.1:12345/-/ready
```

An HTTP 404 response from the Loki root URL `/` does not by itself indicate a failure. Use the documented readiness endpoint or Grafana's data-source test.

If logs do not appear, check Alloy configuration, file permissions, Docker socket access, Loki connectivity, labels, and the selected time range.

## Security Considerations

- Do not commit passwords, tokens, private keys, or other secrets.
- Do not expose Loki's unauthenticated API directly to the public internet.
- Restrict access to Grafana and its administrative account.
- Protect access to `/var/run/docker.sock`; access to the Docker socket can grant powerful host-level privileges.
- Limit access to system logs, which may contain usernames, IP addresses, and other sensitive operational information.
- Review firewall rules before exposing service ports.
- Set an appropriate retention period and monitor disk usage.

## Limitations and Future Improvements

Possible next steps include:

- Monitoring additional Linux servers
- Collecting application logs from other services
- Integrating Issabel/Asterisk logs
- Configuring alerting
- Adding dashboards for application-specific errors
- Defining backups and log-retention policies
- Adding automated deployment and configuration validation

## License

Choose a license appropriate for your intended use before publishing this project.
