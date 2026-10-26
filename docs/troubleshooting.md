
# Troubleshooting Guide

This document covers common troubleshooting steps for the Grafana, Loki, and Alloy logging stack.

## 1. Check Docker Containers

List running containers:

```bash
docker ps
```

Check the status of the Compose project:

```bash
docker compose ps
```

## 2. Inspect Service Logs

Check Loki:

```bash
docker logs --tail=100 loki
```

Check Grafana:

```bash
docker logs --tail=100 grafana
```

Check Alloy:

```bash
docker logs --tail=100 alloy
```

## 3. Check Loki Readiness

On the Docker host, run:

```bash
curl -i http://127.0.0.1:3100/ready
```

A successful readiness response indicates that Loki is ready to serve requests.

An HTTP 404 response from the root URL `/` alone does not mean Loki is broken.

## 4. Check Alloy Readiness

If Alloy's HTTP endpoint is enabled on localhost:

```bash
curl -i http://127.0.0.1:12345/-/ready
```

Review Alloy logs if the readiness check fails.

## 5. Verify the Grafana Data Source

1. Open Grafana.
2. Navigate to Connections and Data sources.
3. Open the Loki data source.
4. Verify its URL.
5. Use Save & test.

When Grafana and Loki share a Docker network, the Loki service name can be used:

```text
http://loki:3100
```

Use an address that is reachable from the Grafana container.

## 6. Logs Are Missing

Check the following:

- Alloy is running.
- The Alloy configuration is valid.
- Alloy can reach Loki.
- The required log files exist on the host.
- The Alloy container can read those files.
- Alloy has access to the Docker socket when collecting container logs.
- The selected Grafana time range includes the log timestamps.
- LogQL selectors match the labels emitted by Alloy.

Example system log query:

```logql
{job="system", host="odooo"}
```

Example Docker log query:

```logql
{job="docker", host="odooo"}
```

## 7. Docker Logs Are Missing

Check that the Docker socket is mounted into Alloy:

```text
/var/run/docker.sock
```

Access to this socket is highly privileged. Do not expose it to untrusted users or containers.

## 8. Dashboard Panels Are Empty

Open Grafana Explore and run the same query used by the panel.

If the query returns no results, verify the actual labels before changing the dashboard.

Use the Explore label browser to inspect available labels and values.

## 9. Disk Usage and Retention

Loki retention is configured in the Loki configuration file.

Review the retention policy and monitor available disk space regularly. Retention settings do not replace backups or capacity planning.

## 10. Security Checklist

- Do not commit passwords, tokens, or private keys.
- Do not expose Loki's unauthenticated API directly to the public internet.
- Restrict access to Grafana.
- Protect access to the Docker socket.
- Review firewall rules and exposed ports.
- Avoid publishing production logs in public repositories.

## 11. Before Opening an Issue

Include:

- Docker container status
- Relevant error messages with secrets removed
- The affected component
- The relevant configuration with sensitive values redacted
- Steps to reproduce the problem

Never attach passwords, tokens, private keys, or sensitive production logs.
