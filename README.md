# Observability Platform

A complete observability solution integrating **Grafana**, **Prometheus**, **Loki**, and **Mimir** for comprehensive metrics and log monitoring on macOS.

![Observability Platform Architecture](/.gemini/antigravity/brain/15eb1fec-0b27-4f82-9b13-559b07950d08/observability_architecture_1764044009766.png)

## Overview

This platform provides end-to-end observability with:
- **Real-time Metrics** - Prometheus for immediate system insights
- **Long-term Metrics Storage** - Mimir for historical data and trend analysis
- **Log Aggregation** - Loki for centralized log management
- **Unified Visualization** - Grafana dashboards combining metrics and logs

## Architecture

### Components

| Component | Version | Port | Purpose |
|-----------|---------|------|---------|
| **Grafana** | 12.3.0 | 3000 | Unified visualization and dashboards |
| **Prometheus** | 3.7.3 | 9090 | Real-time metrics collection |
| **Loki** | 3.6.1 | 3100 | Log aggregation and querying |
| **Promtail** | 3.6.1 | 9080 | Log shipping agent |
| **Mimir** | 2.17.2 | 9009 | Long-term metrics storage |

### Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                       Data Sources                          │
│  ┌──────────────────┐          ┌─────────────────────┐     │
│  │ System Metrics   │          │  Application Logs   │     │
│  │ (node_exporter)  │          │  (System Logs)      │     │
│  └────────┬─────────┘          └──────────┬──────────┘     │
└───────────┼────────────────────────────────┼────────────────┘
            │                                │
            │ scrape                         │ ship logs
            ▼                                ▼
┌───────────────────────────────────────────────────────────┐
│               Collection & Storage                         │
│  ┌─────────────┐    ┌──────────┐    ┌──────────────┐     │
│  │ Prometheus  │───▶│  Mimir   │    │     Loki     │     │
│  │ Port 9090   │    │Port 9009 │    │  Port 3100   │     │
│  └──────┬──────┘    └────┬─────┘    └──────┬───────┘     │
└─────────┼────────────────┼─────────────────┼──────────────┘
          │                │                 │
          │ query          │ query           │ query
          └────────────────┴─────────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Grafana   │
                    │  Port 3000  │
                    └─────────────┘
```

## Quick Start

### Access Grafana

Open your browser and navigate to:
```
http://localhost:3000
```

Default credentials (if not changed):
- **Username:** `admin`
- **Password:** `admin`

### Verify Services

Check all services are running:
```bash
# Homebrew services
brew services list | grep -E '(grafana|prometheus|loki|promtail)'

# Mimir service
ps aux | grep mimir | grep -v grep
```

## Data Sources

Three data sources are pre-configured in Grafana:

### 1. Prometheus (Default)
- **URL:** `http://localhost:9090`
- **Use Case:** Real-time metrics, last 15 days of data
- **Example Query:** `up{job="node_exporter"}`

### 2. Loki
- **URL:** `http://localhost:3100`
- **Use Case:** Log queries and exploration
- **Example Query:** `{job="varlogs"}`

### 3. Mimir
- **URL:** `http://localhost:9009/prometheus`
- **Use Case:** Long-term metrics storage and historical queries
- **Example Query:** `up{cluster="local-mac"}`

## Usage Examples

### Query Metrics

**From Prometheus (Real-time):**
```promql
# Check service uptime
up{job="node_exporter"}

# CPU usage
rate(node_cpu_seconds_total[5m])
```

**From Mimir (Historical):**
```promql
# Historical uptime with cluster label
up{cluster="local-mac"}

# Long-term trend analysis
rate(node_cpu_seconds_total{cluster="local-mac"}[1h])
```

### Query Logs

**From Loki:**
```logql
# All system logs
{job="varlogs"}

# Filter by filename
{filename="/var/log/system.log"}

# Search for specific text
{job="varlogs"} |= "error"
```

## Service Management

### Start/Stop Services

**Homebrew Services (Grafana, Prometheus, Loki, Promtail):**
```bash
# Start
brew services start grafana
brew services start prometheus
brew services start loki
brew services start promtail

# Stop
brew services stop grafana
brew services stop prometheus
brew services stop loki
brew services stop promtail

# Restart
brew services restart <service-name>
```

**Mimir:**
```bash
# Start
launchctl load ~/Library/LaunchAgents/com.grafana.mimir.plist

# Stop
launchctl unload ~/Library/LaunchAgents/com.grafana.mimir.plist

# Restart
launchctl unload ~/Library/LaunchAgents/com.grafana.mimir.plist
launchctl load ~/Library/LaunchAgents/com.grafana.mimir.plist
```

### View Logs

**Service Logs:**
```bash
# Grafana
tail -f /opt/homebrew/var/log/grafana/grafana.log

# Prometheus
tail -f /opt/homebrew/var/log/prometheus/prometheus.log

# Mimir
tail -f /Users/reginravi/Documents/Grafana_Mirmir_loki/mimir-stdout.log
tail -f /Users/reginravi/Documents/Grafana_Mirmir_loki/mimir-stderr.log
```

## File Locations

### Configuration Files

```
/opt/homebrew/etc/
├── loki-local-config.yaml              # Loki configuration
├── promtail-local-config.yaml          # Promtail configuration
├── prometheus.yml                       # Prometheus configuration
└── grafana/
    └── provisioning/
        └── datasources/
            └── observability.yaml       # Grafana data sources

/Users/reginravi/Documents/Grafana_Mirmir_loki/
├── mimir-config.yaml                    # Mimir configuration
└── start-mimir.sh                       # Mimir startup script
```

### Data Storage

```
/opt/homebrew/var/
├── loki/                                # Loki data
└── lib/grafana/                         # Grafana data

/Users/reginravi/Documents/Grafana_Mirmir_loki/
└── mimir-data/                          # Mimir data
    ├── tsdb/
    ├── blocks/
    ├── compactor/
    ├── rules/
    └── alertmanager/
```

## API Endpoints

### Health Checks

```bash
# Grafana
curl http://localhost:3000/api/health

# Prometheus
curl http://localhost:9090/-/healthy

# Loki
curl http://localhost:3100/ready

# Mimir
curl http://localhost:9009/ready
```

### Query APIs

**Prometheus:**
```bash
curl "http://localhost:9090/api/v1/query?query=up"
```

**Mimir:**
```bash
curl -H "X-Scope-OrgID: demo" "http://localhost:9009/prometheus/api/v1/query?query=up"
```

**Loki:**
```bash
curl "http://localhost:3100/loki/api/v1/labels"
```

## Key Features

✅ **Unified Visualization** - Single Grafana instance for all observability data  
✅ **Dual Metrics Storage** - Prometheus for real-time, Mimir for long-term retention  
✅ **Centralized Logs** - Loki aggregates logs from all sources  
✅ **Multi-tenancy Support** - Mimir configured with tenant ID for isolation  
✅ **Auto-start Services** - All services start automatically on system boot  
✅ **Prometheus Remote Write** - Automatic metrics forwarding to Mimir

## Troubleshooting

### Services Won't Start

Check if ports are already in use:
```bash
lsof -i :3000  # Grafana
lsof -i :9090  # Prometheus
lsof -i :3100  # Loki
lsof -i :9009  # Mimir
```

### No Data in Grafana

1. Wait 15-30 seconds for services to initialize
2. Check Prometheus targets: `http://localhost:9090/targets`
3. Verify remote_write status: `http://localhost:9090/tsdb-status`
4. Check service logs for errors

### Mimir "no org id" Error

Ensure the `X-Scope-OrgID` header is set:
- In queries: `-H "X-Scope-OrgID: demo"`
- In Grafana data source configuration (already configured)

### Loki Not Receiving Logs

Check Promtail is running and configured:
```bash
brew services list | grep promtail
cat /opt/homebrew/etc/promtail-local-config.yaml
```

## Next Steps

### Create Dashboards

1. Navigate to Grafana → Dashboards → New Dashboard
2. Add panels with queries from any data source
3. Combine metrics and logs for comprehensive monitoring

### Configure Alerts

1. Set up Prometheus alerting rules
2. Configure Mimir ruler for long-term alerting
3. Integrate with Alertmanager

### Extend Data Collection

1. Add custom metrics exporters
2. Configure additional log sources in Promtail
3. Create custom instrumentation for applications

### Security Hardening

1. Enable authentication on all services
2. Configure HTTPS/TLS
3. Set up proper access controls and RBAC
4. Change default credentials

## Architecture Benefits

- **Scalability** - Mimir designed for long-term, scalable storage
- **Separation of Concerns** - Hot data in Prometheus, cold data in Mimir
- **Cost Efficiency** - Optimize storage costs with tiered architecture
- **Query Performance** - Fast queries on recent data, historical queries on Mimir
- **Unified View** - Single pane of glass for metrics and logs

## Technical Specifications

### Prometheus Configuration

**Remote Write:**
- Endpoint: `http://localhost:9009/api/v1/push`
- Tenant: `demo`
- Queue Capacity: 10,000 samples
- Max Shards: 10

**External Labels:**
- `cluster: local-mac`
- `environment: development`

### Mimir Configuration

**Mode:** Single-process (all-in-one)  
**Storage:** Filesystem  
**Retention:** Configurable (default: unlimited)  
**Tenancy:** Multi-tenant with ID `demo`

### Loki Configuration

**Storage:** Filesystem (`/opt/homebrew/var/loki`)  
**Schema:** v13 with TSDB  
**Ingestion:** Via Promtail

## Support

For issues or questions:
- Check service logs
- Verify configuration files
- Test API endpoints
- Review the [walkthrough documentation](/.gemini/antigravity/brain/15eb1fec-0b27-4f82-9b13-559b07950d08/walkthrough.md)

## Version Information

- **Platform:** macOS (ARM64)
- **Installation Date:** 2025-11-25
- **Last Updated:** 2025-11-25

---

**Status:** ✅ All services operational

Access your observability platform at **http://localhost:3000**
