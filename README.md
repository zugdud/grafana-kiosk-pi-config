# Grafana Kiosk Pi Setup

## Services Overview

#### 1. **Grafana Server** (`grafana-server.service`)
- **Purpose**: Main monitoring and visualization platform
- **Description**: Runs the Grafana web interface for creating and viewing dashboards
- **Dependencies**: Requires PostgreSQL, MariaDB, MySQL, or InfluxDB
- **User**: Runs as `grafana` user with enhanced security settings
- **Port**: Default HTTP port (typically 3000)

#### 2. **Prometheus** (`prometheus.service`)
- **Purpose**: Time-series database for metrics collection
- **Description**: Collects and stores metrics from various exporters
- **User**: Runs as `pi` user
- **Config**: Uses `/home/pi/prometheus/prometheus.yml`
- **Storage**: Data stored in `/home/pi/prometheus/data`

### Data Exporters

#### 3. **Flume Water Exporter** (`flume-exporter.service`)
- **Purpose**: Exports water usage data from Flume devices
- **Description**: Collects water consumption metrics and exposes them to Prometheus
- **User**: Runs as `pi` user
- **Config**: Uses environment file `/etc/flume-exporter/config.env`
- **Features**: Secure configuration with environment variables for API credentials

#### 4. **LibreHardwareMonitor Exporter** (`librehardwaremonitor-exporter.service`)
- **Purpose**: Exports hardware monitoring metrics
- **Description**: Collects system performance data (CPU, GPU, memory, etc.)
- **User**: Runs as `librehardwaremonitor` user
- **Working Directory**: `/opt/librehardwaremonitor-exporter`

#### 5. **Pi-hole Exporter** (`pihole-exporter.service`)
- **Purpose**: Exports Pi-hole DNS statistics
- **Description**: Collects DNS query data, blocked domains, and Pi-hole performance metrics
- **User**: Runs as `pihole-exporter` user
- **Target**: Connects to Pi-hole at `<PIHOLE_IP>`
- **Security**: Uses API token authentication with enhanced security settings

### Display and UI

#### 6. **Grafana Kiosk** (`grafana-kiosk.service`)
- **Purpose**: Full-screen dashboard display for TV/monitor
- **Description**: Automatically displays a specific Grafana dashboard in kiosk mode
- **User**: Runs as `pi` user
- **Display**: Uses X11 display `:0`
- **Features**: 
  - Disables screensaver and monitor standby
  - Auto-refreshes every 30 seconds
  - Shows dashboard: `zugstats` (ID: 9162e1e2-4b21-489c-868b-37c96f44db30)
  - Time range: Last 30 minutes

#### 7. **Unclutter** (`unclutter.service`)
- **Purpose**: Hides mouse cursor for clean kiosk display
- **Description**: Automatically hides cursor after 0.5 seconds of inactivity
- **User**: Runs as `pi` user
- **Display**: Works with X11 display `:0`

## Installation and Setup

### Prerequisites
- Raspberry Pi with Raspberry Pi OS
- PostgreSQL, MariaDB, MySQL, or InfluxDB (for Grafana)
- X11 display environment (for kiosk mode)

### Service Installation
1. Copy systemd service files to `/etc/systemd/system/`
2. Copy configuration files to appropriate locations
3. Enable and start services:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable <service-name>
   sudo systemctl start <service-name>
   ```

### Configuration Files
- **Prometheus**: `/home/pi/prometheus/prometheus.yml`
- **Flume Exporter**: `/etc/flume-exporter/config.env`
- **Pi-hole Exporter**: `/etc/default/pihole-exporter`
- **Grafana Dashboard**: `zugstats.json` - Import this dashboard into Grafana

## Service Dependencies

```
grafana-server → database (PostgreSQL/MySQL/InfluxDB)
grafana-kiosk → grafana-server (port 3000)
unclutter → display-manager
```

## Security Features

- Most services run with restricted privileges
- Private temporary directories and system protection
- User isolation and capability restrictions
- Secure environment variable handling

## Monitoring Dashboard

The kiosk displays a custom dashboard (`zugstats`) that shows:
- Water usage data from Flume
- System hardware metrics
- Pi-hole DNS statistics
- Auto-refreshing every 30 seconds
- 30-minute time window

**Dashboard Setup**: Import the included `zugstats.json` file into Grafana to create the dashboard that the kiosk will display.

## Security Notes

⚠️ **Important**: This repository contains placeholder values that must be replaced with your actual configuration:

- **Pi-hole Exporter**: Replace `<PIHOLE_IP>` and `<YOUR_PIHOLE_API_TOKEN>` with your actual Pi-hole IP address and API token
- **Flume Exporter**: Replace `placeholder` values in `config/flume-exporter.env` with your actual Flume API credentials
- **Grafana Kiosk**: Replace `placeholder` username/password with your actual Grafana credentials
- **Dashboard**: The `zugstats.json` dashboard contains placeholder device names, IPs, and device IDs that should be updated for your network:
  - Replace `<NETWORK_DEVICE_NAME>` with your actual network device name
  - Replace `<GAMING_PC_NAME>` with your gaming PC name
  - Replace `<FLUME_DEVICE_ID>` with your actual Flume water meter device ID
  - Replace `<TIMESTAMP>` with your preferred timestamp

**Never commit real credentials or sensitive information to version control!**

## Troubleshooting

- Check service status: `sudo systemctl status <service-name>`
- View logs: `sudo journalctl -u <service-name>`
- Verify network connectivity for exporters
- Check X11 display configuration for kiosk mode