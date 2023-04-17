# Purple Air Exporter
A Prometheus extractor for PurpleAir sensor data using the PurpleAir API. This script queries the PurpleAir API (api.purpleair.com) for public sensor(s) and captures the data as Prometheus metrics. Most other implementations of similar extractors I found were using outdated methods of gathering this data (the old /json endpoint was shut down in 2022, and the map.purpleair.com method feels like a hack. Additionally, the official API offers an endpoint to query multiple sensors in a single request.

[![Docker Image Version](https://img.shields.io/docker/v/nirajsanghvi/purpleair_exporter?sort=semver)][hub]
[![Docker Image Size](https://img.shields.io/docker/image-size/nirajsanghvi/purpleair_exporter)][hub]

[hub]: https://hub.docker.com/r/nirajsanghvi/purpleair_exporter/

The following fields are mapped directly from the API response (with each field containing the `sensor_id` and `name` attached as `label`):
- `last_seen` -> `purpleair_last_seen_seconds`
- `pm2.5` -> `purpleair_pm2_5`
- `pm2.5_10minute` -> `purpleair_pm2_5_10_minute`
- `pm10.0` -> `purpleair_pm10_0`
- `temperature` -> `purpleair_temp_f`
- `pressure` -> `purpleair_pressure`
- `humidity` -> `purpleair_humidity`

Additionally, the AQI is computed for PM2.5 (both from the instant value and the 10-minute average) using this US EPA formula to roughly match what is shown on the PurpleAir map (for some reason the API does not provide this calculation in its response): https://community.purpleair.com/t/how-to-calculate-the-us-epa-pm2-5-aqi/877. The AQI values also are computed using the AQandU conversion as well, and you can use the `conversion` label as a filter depending on whether you want to use that (`conversion="AQandU"`) or not (`conversion="None"`)
- `purpleair_aqi_pm2_5`
- `purpleair_aqi_pm2_5_10_minute`
- `purpleair_aqi_pm10_0`

## Configuration
The following environment variables are used by this script:

### `PAE_SENSOR_IDS` (required)
A comma-separated list of sensor IDs to collect data from. Sensor IDs for public sensors can be found on the [PurpleAir map](https://map.purpleair.com) by clicking on the sensor and viewing the widget code, which should contain a 5 or 6-digit number representing the sensor.

### `PAE_API_READ_KEY` (required)
As a relatively new option, you can visit https://develop.purpleair.com/keys to generate API keys. This script only requires the READ key, not the WRITE one since it is only reading sensor data. Note: That website only offers a Google login option, so if you don't want to do that you might have luck trying the older method of simply sending an email to contact@purpleair.com with your first name, last name, and email address to assign them to.

### `PAE_LOGGING` (optional)
Log level of the script, defaults to 'info', accepts any [Python logging level name](https://docs.python.org/3/howto/logging.html#logging-levels)

### `PAE_PROM_PORT` (optional)
The port to run the Prometheus HTTP server on, defaults to 9101

## Docker CLI

```
docker run -d \
  --name=purpleair_exporter \
  -p 9101:9101 \
  -e PAE_SENSOR_IDS=<sensor1>,<sensor2>,<sensor3> \
  -e PAE_API_READ_KEY=<PurpleAir API Read Key> \
  --restart unless-stopped \
  nirajsanghvi/purpleair_exporter
```

## Docker Compose

```yaml
version: '3.9'
services:
  purpleair-exporter:
    image: nirajsanghvi/purpleair_exporter:latest
    environment:
      - 'PAE_SENSOR_IDS=<sensor1>,<sensor2>,<sensor3>'
      - 'PAE_API_READ_KEY=<PurpleAir API Read Key>'
    container_name: PurpleAir_Exporter
    hostname: purpleair_exporter
    ports:
      - 9101:9101
    restart: on-failure:5
```

LICENSE
======

MIT. See `LICENSE.txt`.

    Copyright (c) 2023 Niraj Sanghvi
