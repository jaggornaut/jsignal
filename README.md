# JSignal

Self-hosted radio signal tracking. Captures signals with an SDR dongle, streams them over MQTT, records them to PostgreSQL and shows them live or as replayable history in a desktop console. 

```
SDR dongle
   │
   ▼
adsb-module ──MQTT──→ Mosquitto ──→ jsignal-be ──→ PostgreSQL
                          │                            │
           jsignal-fe ◄───┘   live      history   ◄────┘ HTTP
```

## Repositories

| Repository | Role |
|------------|------|
| [adsb-module](https://github.com/jaggornaut/adsb-module) | Captures and decodes ADS-B at 1090 MHz, publishes JSON to MQTT |
| [jsignal-be](https://github.com/jaggornaut/jsignal-be) | Records MQTT traffic to PostgreSQL, serves history over REST |
| [jsignal-fe](https://github.com/jaggornaut/jsignal-fe) | Desktop console: live map and list, history playback |

## Quick start

This repo ships a compose file that starts the server stack: MQTT broker, PostgreSQL and the recorder backend.

```bash
git clone https://github.com/jaggornaut/jsignal.git
cd jsignal
docker compose up -d
```

MQTT broker on `1883`, history API on `http://localhost:8080` (`curl http://localhost:8080/health`). Database and schema are created on first start.

Then:

1. **Feed it data**: run [adsb-module](https://github.com/jaggornaut/adsb-module) on the machine with the dongle, with `broker_address` pointing at this host (the same machine works too).
2. **Watch it**: run [jsignal-fe](https://github.com/jaggornaut/jsignal-fe) and add two sources from the sidebar, the MQTT broker (`<host>:1883`) for live and the backend (`http://<host>:8080`) for history.

To check traffic without the frontend:

```bash
mosquitto_sub -h localhost -t 'adsb/#' -v
```

## Notes

- The bundled `mosquitto.conf` allows anonymous connections. Fine on a LAN, harden it before exposing further.
- Each component also runs standalone; see the individual READMEs for native setup, configuration and API docs.
