# Winter Haven Chain of Lakes Water Level Monitor

**A Real-Time Bridge Clearance & Water Level Information System**

Ingram Leedy | ingramleedy@gmail.com | Lake Cannon, Winter Haven, FL

---

## About This Project

The Winter Haven Chain of Lakes Water Level Monitor is an IoT-powered system that provides real-time water level data, bridge clearance measurements, and canal depth information for the Winter Haven Chain of Lakes in Polk County, Florida. The system covers the interconnected 9,000-acre chain of 25+ lakes, serving boaters, dock owners, beachfront residents, and the broader community.

Information is published automatically to Instagram (@winterhavenchain) as daily Stories, weekly carousel infographics, and AI-narrated video Reels.

---

## Data Sources & Methodology

### Real-Time IoT Sensor (Southern Chain)

A submersible pressure-based water level sensor is installed at my dock on Lake Cannon. It reports hourly readings via cellular (LTE-M) to Azure IoT Hub.

| Component | Details |
|-----------|---------|
| Water Level Probe | ULB16 pressure sensor (0–5m range, 0.25% accuracy) |
| Temperature Probe | STT-26 submersible (0–35°C range) |
| Communication | RAK2560 hub with LTE-M cellular (Soracom) |
| Power | 12V DC + solar battery backup (5200 mAh) |
| Reporting Interval | Approximately every hour |
| Location | Lake Cannon dock (28.03667, -81.74719) |

**Calibration:** The sensor outputs a voltage proportional to water depth. A two-point calibration (dry = 4.13V / 0 ft, mounted = 6.71V / 2.83 ft) converts voltage to depth. The sensor's known bottom elevation (127.5647 ft NAVD88) translates depth readings into surface elevation above sea level.

### SWFWMD SCADA Data (Northern Chain)

Daily mean water level readings for the Northern Chain are sourced from the Southwest Florida Water Management District's (SWFWMD) automated SCADA gauge at Lake Smart (Station 772960) via the KiWIS API. This provides the daily mean water level in NAVD88.

### Bridge Heights & Canal Depths

All bridge heights and canal bottom depths were physically measured and recorded as elevations in the NAVD88 vertical datum (feet above sea level). This allows real-time computation of:

- **Bridge Clearance** = Bridge Height (NAVD88) − Current Water Level (NAVD88)
- **Canal Water Depth** = Current Water Level (NAVD88) − Canal Bottom Elevation (NAVD88)

### Historical Context & Base Datum

The SWFWMD staff gauge on Lake Cannon (Station 24852, located at 28.04125, -81.74747) is a manually-read gauge that SWFWMD visits approximately monthly to record the lake's surface elevation in NAVD88. This station's records were critical to the project in two ways:

1. **Base datum for sensor calibration** — The staff gauge's NAVD88 readings were used to establish the known bottom elevation of our IoT sensor (127.5647 ft NAVD88), which is the reference point for converting raw voltage readings into surface elevations.
2. **Historical data** — Years of staff gauge records predate our IoT sensor installation, providing the long-term dataset used to calculate all-time high/low water levels, long-term median, and 7-day/30-day trend analysis.

### Weather Data

A WeatherFlow Tempest weather station (Station 26857) is installed at the same dock location as the IoT water level sensor. The Tempest is an all-in-one personal weather station that measures wind speed/direction, air temperature, humidity, barometric pressure, UV index, solar radiation, and precipitation.

The system uses the WeatherFlow REST API to pull two types of data for the infographics:

- **Current observations** — Real-time air temperature, wind speed and direction, UV index, and sky conditions displayed alongside our water temperature reading
- **Daily forecast** — Today's high/low temperatures, conditions summary, and precipitation probability

This gives viewers a complete picture — water conditions from our sensor, plus weather context from the same location — all in one infographic.

---

## Coverage

### Southern Chain — 10 Bridges + 4 Bridgeless Canals

| Canal | Bridge | Canal Depth |
|-------|--------|-------------|
| Winterset/Eloise | Yes | Yes |
| Eloise/Lulu | Yes | Yes |
| Lulu/Shipp | Yes | Yes |
| Lulu/Roy | Yes | Yes |
| Shipp/May | Yes | Yes |
| May/Howard | Yes | Yes |
| Howard/Cannon | Yes | Yes |
| Mirror/Cannon | Yes | Yes |
| Idylwild/Cannon | Yes | Yes |
| Hartridge/Idylwild | Yes | Yes |
| Summit/Eloise | No bridge | Depth only |
| Winterset/Little Winterset | No bridge | Depth only |
| Mirror/Spring | No bridge | Depth only |
| Idylwild/Jessie | No bridge | Depth only |

### Northern Chain — 4 Bridges

| Canal | Bridge | Canal Depth |
|-------|--------|-------------|
| Haines/Rochelle | Yes | Yes |
| Rochelle/Conine | Yes | Yes |
| Smart/Conine | Yes | Yes |
| Conine/Hartridge | Yes | Yes |

---

## What Gets Published

### Daily Instagram Stories (Every Morning, 9 AM EST)

Two Stories posted daily — one for each chain — showing:

- Current water level (ft NAVD88) with trend arrow
- 7-day and 30-day water level change
- Current conditions (air temp, water temp, wind, UV)
- Today's forecast (high/low, conditions, rain chance)
- Historical context (all-time high/low, median, position)
- Bridge clearance table with per-bridge clearance and canal depth
- Low-water warnings when levels drop more than 6 inches below median

### Weekly Carousel Post (Fridays, 10 AM EST)

A swipeable Instagram carousel with both Southern and Northern Chain infographics in feed-optimized format (1080×1350).

### Weekly Video Reel (Fridays, 11 AM EST)

An AI-narrated 22+ second video Reel featuring:

- AI-generated hook headline highlighting the week's key statistic
- Data slides for both chains
- Call-to-action with account follow prompt
- Background music and burned-in captions (for sound-off viewing)
- Dynamic narration that adjusts for season, weather, and water conditions

---

## System Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌───────────────────┐
│  Water Level     │────▶│  Azure       │────▶│  ProcessIoTMessage │
│  Sensor (LTE-M) │     │  IoT Hub     │     │  (Azure Function)  │
└─────────────────┘     └──────────────┘     └────────┬──────────┘
                                                       │
┌─────────────────┐     ┌──────────────┐              ▼
│  SWFWMD KiWIS   │────▶│ FetchLake-   │────▶┌───────────────────┐
│  API (SCADA)    │     │ Readings     │     │  Azure SQL Server  │
└─────────────────┘     └──────────────┘     │  (Telemetry,       │
                                              │   BridgeClearances,│
┌─────────────────┐                           │   Historical)      │
│  WeatherFlow    │──┐                        └────────┬──────────┘
│  Tempest API    │  │                                 │
└─────────────────┘  │  ┌──────────────────────────────┘
                     │  │
                     ▼  ▼
              ┌──────────────────┐     ┌───────────────┐
              │  Image & Video   │────▶│  Azure Blob   │
              │  Generation      │     │  Storage      │
              │  (Azure Funcs)   │     └───────┬───────┘
              └──────────────────┘             │
                                               ▼
                                        ┌─────────────┐
                                        │  Instagram   │
                                        │  Graph API   │
                                        └─────────────┘
```

**All infrastructure runs on Microsoft Azure:**

| Resource | Purpose |
|----------|---------|
| Azure IoT Hub | Receives sensor telemetry via MQTT |
| Azure SQL Database | Stores all water level data, bridge reference data, historical records |
| Azure Functions (Python) | 11 serverless functions for data processing, image generation, posting |
| Azure Blob Storage | Temporary storage for generated images and videos |
| Azure Communication Services | Email alerts for data health monitoring |

Authentication uses Azure Managed Identity (no passwords stored in code).

---

## Available Data

The SQL database contains structured, queryable data accessible via stored procedures:

| Data | Description | Update Frequency |
|------|-------------|-----------------|
| Current water level | Surface elevation (ft NAVD88) for both chains | Hourly (Southern), Daily (Northern) |
| Water temperature | Lake Cannon water temperature (°F/°C) | Hourly |
| Bridge clearances | Real-time clearance at each bridge (ft + in) | Computed on demand |
| Canal water depths | Current depth at each canal crossing | Computed on demand |
| 7-day / 30-day trends | Water level change over time | Computed on demand |
| Historical stats | All-time high, low, median, date of occurrence | Updated with each reading |
| Data health status | Last reading time, staleness alerts | Monitored every 4 hours |

### Vertical Datum

All elevation measurements use **NAVD88** (North American Vertical Datum of 1988), the standard vertical datum used by SWFWMD and USGS, ensuring compatibility with other government elevation data.

---

## Collaboration Opportunities

This system was designed to be extensible. Possible ways to share or integrate the data:

- **Custom API endpoint** — A dedicated HTTP endpoint returning JSON data (current levels, clearances, depths, trends) for use in web apps or GIS systems
- **Scheduled data export** — Automated daily/hourly data drops to a city-hosted location
- **Embeddable widget** — A lightweight web component showing current conditions
- **GIS integration** — Bridge and canal data includes latitude/longitude coordinates, compatible with ArcGIS, QGIS, or web mapping platforms
- **Raw data feed** — Direct SQL view or CSV export for city analysts

I'm happy to work with the City's technical team to design whatever integration format works best for your website and geo apps.

---

## About

This project is a personal initiative by Ingram Leedy, a resident on Lake Cannon in Winter Haven. What started as curiosity about water levels at my own dock grew into a comprehensive monitoring system serving the boating community across both the Southern and Northern Chains of Lakes.

The entire system — from the physical sensor installation to the cloud data pipeline to the AI-generated Instagram content — is designed, built, and maintained as a passion project.

**Contact:** Ingram Leedy — ingramleedy@gmail.com

**Instagram:** @winterhavenchain

---

*All bridge clearance and canal depth measurements are estimates based on sensor readings and physical measurements. They should be used for informational purposes only and not as a substitute for safe boating practices. Always verify conditions visually before navigating under bridges or through canals.*
