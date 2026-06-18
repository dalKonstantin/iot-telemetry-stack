# IoT Telemetry Stack

End-to-end stack for collecting ESP32 sensor telemetry over MQTT, storing it in MongoDB, and viewing it in a web dashboard.

## Architecture

```
ESP32 (BME280) ──MQTT──▶ Mosquitto ──▶ telemetry-gateway ──▶ MongoDB
                                              │
                                              └── HTTP API ──▶ telemetry-gateway-frontend
```

| Component | Description |
|-----------|-------------|
| [esp32-mqtt-telemetry](esp32-mqtt-telemetry/) | ESP32 firmware — reads BME280, publishes JSON to MQTT |
| [telemetry-gateway](telemetry-gateway/) | Go service — subscribes to MQTT, stores telemetry, exposes REST API |
| [telemetry-gateway-frontend](telemetry-gateway-frontend/) | React dashboard — live device telemetry viewer |

## Quick start (Docker)

Requires Docker and Docker Compose.

```bash
git clone --recurse-submodules git@github.com:dalKonstantin/iot-telemetry-stack.git
cd iot-telemetry-stack
docker compose up -d --build
```

| Service | URL / port |
|---------|------------|
| Dashboard | http://localhost:5173 |
| API | http://localhost:8080 |
| MQTT broker | `localhost:1883` |
| MongoDB | `localhost:27017` |

```bash
docker compose logs -f          # follow logs
docker compose down             # stop containers
docker compose down -v          # stop and remove MongoDB data
```

Point your ESP32 at the MQTT broker (`mqtt://<host-ip>:1883`, topic `esp32-telemetry/telemetry`). See [esp32-mqtt-telemetry](esp32-mqtt-telemetry/) for firmware setup.

## API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/devices` | List known device IDs |
| `GET` | `/api/devices/{device}/telemetry/latest` | Latest telemetry for a device |

## Repository layout

This repo uses git submodules. After cloning without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

```
iot-telemetry-stack/
├── docker-compose.yml              # full stack (MongoDB, Mosquitto, gateway, frontend)
├── esp32-mqtt-telemetry/           # ESP32 firmware
├── telemetry-gateway/              # Go MQTT gateway + REST API
└── telemetry-gateway-frontend/     # React dashboard
```

## Local development

Run infrastructure and services separately when developing individual components.

**Infrastructure** (MongoDB + Mosquitto):

```bash
cd telemetry-gateway
docker compose up -d
```

**Gateway** (from `telemetry-gateway/`):

```bash
cp configs/config.toml.example configs/config.toml
go run ./src/cmd --config configs/config.toml
```

**Frontend** (from `telemetry-gateway-frontend/`):

```bash
npm install
npm run dev
```

Open http://localhost:5173. The Vite dev server proxies `/api` to the gateway on port 8080.

See each submodule's README for more detail.
