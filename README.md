# Van IoT Monitoring System

This repository contains the Docker setup for a complete van monitoring system using MQTT, InfluxDB, and Telegraf. The system collects data from an Alpicool refrigerator and a Bluetti power station via ESP32-based Bluetooth to MQTT bridges.

## Architecture

The system consists of:

1. **EMQX MQTT Broker**: Handles all MQTT communication between sensors and the data pipeline
2. **InfluxDB**: Time-series database for storing all sensor data
3. **Telegraf**: Data collection agent that subscribes to MQTT topics and writes to InfluxDB
4. **ESP32 Bridges**: Two ESP32-based Bluetooth to MQTT bridges:
   - Bluetti ESP32 Bridge: Connects to Bluetti power stations and publishes data to MQTT
   - Alpicool ESP32 MQTT Bridge: Connects to Alpicool refrigerators and publishes data to MQTT

## Prerequisites

- Docker and Docker Compose
- Git
- ESP32 development environment (if modifying the bridge code)
- Two ESP32 devices (one for each bridge)

## Directory Structure

```
van-iot-monitoring/
├── docker-compose.yml         # Docker Compose configuration
├── telegraf/                  # Telegraf configuration
│   ├── telem.toml             # Telegraf configuration file
│   └── debug-alpicool.toml    # Debug configuration for Alpicool
├── influxdb-config/           # InfluxDB configuration and dashboards
│   └── dashboards/            # Dashboard exports
└── scripts/                   # Utility scripts
    └── backup-dashboards.sh   # Script to back up InfluxDB dashboards
```

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/van-iot-monitoring.git
cd van-iot-monitoring
```

### 2. Clone the ESP32 Bridge Repositories

```bash
# Create ESP32 directory
mkdir -p esp32
cd esp32

# For the Bluetti power station monitor
git clone https://github.com/mariolukas/Bluetti_ESP32_Bridge.git bluetti

# For the Alpicool refrigerator monitor
git clone https://github.com/jakub-hajek/alpicool-esp32-mqtt.git alpicool

cd ..
```

### 3. Configure Environment Variables

Edit the `docker-compose.yml` file to update passwords and tokens as needed:

```yaml
DOCKER_INFLUXDB_INIT_PASSWORD=your_secure_password
DOCKER_INFLUXDB_INIT_ORG=your_organization
DOCKER_INFLUXDB_INIT_BUCKET=van
DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=your_influxdb_token
```

Make sure to update the same token in the `INFLUX_TOKEN` environment variable for the Telegraf service.

### 4. Start the Services

```bash
docker-compose up -d
```

## ESP32 Bridge Setup

### Bluetti ESP32 Bridge

1. Open the Bluetti ESP32 Bridge project in your ESP32 development environment
2. Follow the instructions in the repository's README to configure:
   - WiFi settings
   - MQTT broker address (set to the IP address of your EMQX instance)
   - Bluetti device settings
3. Flash the firmware to your first ESP32 device

### Alpicool ESP32 MQTT Bridge

1. Open the Alpicool ESP32 MQTT Bridge project in your ESP32 development environment
2. Follow the instructions in the repository's README to configure:
   - WiFi settings
   - MQTT broker address (set to the IP address of your EMQX instance)
   - Alpicool device settings
3. Flash the firmware to your second ESP32 device

## InfluxDB Setup

After starting the services, InfluxDB will be available at http://localhost:8086. The initial setup is handled by the Docker Compose environment variables.

To restore dashboards:

1. Navigate to the InfluxDB web interface
2. Go to Dashboards → Import Dashboard
3. Select the dashboard JSON files from the `influxdb-config/dashboards` directory

## MQTT Broker

The EMQX MQTT broker will be available at:
- MQTT Port: 1883
- WebSocket Port: 8083
- Admin Dashboard: http://localhost:18083 (default credentials: admin/public)

## Data Structure

### Alpicool Refrigerator

Data is published to `tele/alpicool/sensor` in JSON format:

```json
{
  "f_voltage": 13.19999981,
  "f_eco": true,
  "f_on": true,
  "f_actual_temperature": 5,
  "f_desired_temperature": 7
}
```

### Bluetti Power Station

Data is published to multiple topics with a base path of `bluetti/AC200L2425000019872/state/`:

```
bluetti/AC200L2425000019872/state/ac_output_mode 1
bluetti/AC200L2425000019872/state/internal_ac_voltage 11.90
bluetti/AC200L2425000019872/state/internal_current_one 13.70
...and many others
```

## Backup and Restore

### Backing Up Dashboards

Run the provided backup script:

```bash
./scripts/backup-dashboards.sh
```

This will export all dashboards to the `influxdb-config/dashboards` directory.

### Backing Up Data

InfluxDB data is stored in a Docker volume. To back up this data:

```bash
docker-compose stop influxdb
docker run --rm -v van-iot-monitoring_influxdb-data:/backup -v $(pwd)/backup:/backup busybox tar -czvf /backup/influxdb-data.tar.gz /backup
docker-compose start influxdb
```

## Troubleshooting

### MQTT Connection Issues

Check if the MQTT broker is accessible:

```bash
mosquitto_sub -h localhost -t 'tele/alpicool/#' -v
```

### Telegraf Issues

Check Telegraf logs:

```bash
docker-compose logs telegraf
```

### ESP32 Bridge Issues

For Bluetti ESP32 Bridge issues, refer to the troubleshooting section in:
https://github.com/mariolukas/Bluetti_ESP32_Bridge

For Alpicool ESP32 MQTT Bridge issues, refer to the troubleshooting section in:
https://github.com/jakub-hajek/alpicool-esp32-mqtt

### Debug Configuration

A debug configuration for Telegraf is provided in `telegraf/debug-alpicool.toml`. To use it:

```bash
docker run --rm --network host -v $(pwd)/telegraf/debug-alpicool.toml:/etc/telegraf/telegraf.conf:ro telegraf:latest
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.
