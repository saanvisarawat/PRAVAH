<div align="center">

# PRAVAH

### AI/ML-Based Heavy Rainfall Early Warning, Inundation Prediction & Flood Response Platform

*A full-stack disaster-response system for Kerala, India: live telemetry, explainable flood-risk prediction, inundation mapping, crowd-verified SOS, volunteer dispatch, and offline-first citizen tools.*

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-backend-009688?logo=fastapi&logoColor=white)
![PostGIS](https://img.shields.io/badge/PostgreSQL-PostGIS-336791?logo=postgresql&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-SHAP-EC6B23)
![Flutter](https://img.shields.io/badge/Flutter-Android%20%7C%20iOS%20%7C%20Web-02569B?logo=flutter&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini-RAG%20%26%20agents-4285F4?logo=google&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)

</div>

---

## Table of Contents

1. [Overview](#1-overview)
2. [Key Features](#2-key-features)
3. [System Architecture](#3-system-architecture)
4. [Machine Learning & Inundation Methodology](#4-machine-learning--inundation-methodology)
5. [Tech Stack](#5-tech-stack)
6. [Repository Structure](#6-repository-structure)
7. [Getting Started](#7-getting-started)
8. [Configuration Reference](#8-configuration-reference)
9. [API Reference](#9-api-reference)
10. [Real-Time Events (WebSocket)](#10-real-time-events-websocket)
11. [Roles & Access Control](#11-roles--access-control)
12. [Offline & Low-Connectivity Design](#12-offline--low-connectivity-design)
13. [Deployment](#13-deployment)
14. [Testing](#14-testing)
15. [Current Limitations & Simulated Components](#15-current-limitations--simulated-components)
16. [Security & Hardening Checklist](#16-security--hardening-checklist)
17. [Roadmap](#17-roadmap)
18. [Contributing](#18-contributing)
19. [License](#19-license)
20. [Acknowledgements](#20-acknowledgements)

---

## 1. Overview

**PRAVAH** (Sanskrit for *flow*) is an end-to-end flood intelligence and response platform built around **Kerala's 14 districts**. It was developed for problem statement **PS 26071: "AI/ML-based integrated heavy rainfall early warning and inundation prediction system"**, and brings three groups of people onto one system:

| Audience | What PRAVAH gives them |
|---|---|
| **Predictors** (IMD meteorologists, MoES analysts) | Live four-pillar telemetry, a district-level flood-risk model with SHAP explanations, inundation maps and time-lapse simulation |
| **Responders** (district authorities, NDMA/NDRF, volunteers) | Approval queue for model-raised alerts, RAG-generated NDMA protocol checklists, SOS ticket queue, volunteer dispatch, flooded-road intelligence |
| **Citizens** | One-tap SOS that works on bad networks, crowd verification of nearby hazards, evacuation map and shelters, a multilingual survival assistant (text and voice) |

### What makes it different

- **Four-pillar data fusion.** Observational (AWS/rain gauge), numerical weather prediction, satellite and Doppler-radar signals feed a single classifier. Where a pillar isn't available it is synthesized with physically motivated relations (e.g. Marshall-Palmer Z-R for radar reflectivity).
- **Explainable by default.** Every prediction can be traced to its drivers through SHAP: global feature importance plus per-district local explanations.
- **Human-in-the-loop alerting.** The model *proposes* a CRITICAL alert; a responder *approves or rejects* it before citizens are notified.
- **Built for disasters, not demos.** Offline SOS queue with idempotent bulk sync, 2G-friendly "lite" SOS endpoint, SMS fallback through a Twilio-compatible webhook, offline map tiles, and an offline-capable RAG fallback when cloud AI is unreachable.

---

## 2. Key Features

### Prediction & early warning
- **Hourly flood-risk pipeline** for all 14 Kerala districts: pulls live weather and river discharge, runs the XGBoost model, and refreshes an in-memory cache served in milliseconds at `GET /api/dashboard/live-kerala`.
- **Four risk classes** (`No Rain`, `Light`, `Moderate`, `Severe`) with a lowered Severe threshold (0.20) and a calibrated **expected inundation depth (cm)** per district.
- **Four-pillar telemetry HUD** per district: INSAT-3DR-style cloud-top temperature, Doppler radar reflectivity (dBZ), AWS ground observations, and a 72-hour NWP accumulation.
- **Reservoir monitoring**: scrapes KSEB/SLDC dam levels (Idukki, Pamba, Sholayar, Idamalayar, Kuttiadi) with a safe fallback.
- **SHAP explainability**: global feature importance and precomputed per-district local drivers (`/api/v1/shap-drivers/{district}`).
- **Inundation mapping**: a pre-rendered 90 m HAND-based depth raster overlay with GPS bounds for map clients, plus a scenario-driven T+0h to T+3h **time-lapse simulation** (`NORMAL` / `EXTREME_EVENT`).
- **Flooded-road intelligence**: when estimated depth reaches the 20 cm vehicle-stall threshold, major roads (motorway, trunk, primary, secondary) intersecting the district are flagged as impassable.

### Response & coordination
- **Human-in-the-loop alert queue**: a transition to `Severe` creates a *pending* alert; responders approve or reject it, and approval broadcasts a `high_risk_alert` to every connected client.
- **Multi-agent Gemini workflow** (Risk Analyst, Resource Allocator, Communications, Coordinator). The allocator is grounded in NDMA guidelines through RAG, and every run is stored with its full execution chain for auditability.
- **NDMA protocol checklists** tiered by alert level (`RED` / `ORANGE` / `GREEN`) alongside IMD advisories.
- **Automatic volunteer dispatch**: a new SOS is matched to an available volunteer within 5 km using PostGIS `ST_DWithin`; officials can manually assign or reassign.
- **Comparative storm analytics**: the current storm's hourly trend against historical benchmarks (Mumbai 2005, Chennai 2015).
- **Masked emergency calls**: FCM high-priority data messages (`EMERGENCY_INCOMING_CALL`) that open a full-screen incoming-call UI for volunteers without exposing the citizen's identity.

### Citizen tools
- **SOS** via the full report flow, the 2G-optimised **lite endpoint**, or an **SMS** to a Twilio-compatible webhook, with abuse throttling.
- **Crowdsourced verification radar**: nearby users vote *Confirm Hazard* or *False Alarm*; three confirmations auto-promote a ticket to **verified**, trigger a real-time broadcast, and (optionally) kick off an n8n/Twilio workflow.
- **RAG survival chatbot** (English and Malayalam) over NDMA and NOAA/FEMA manuals, with a clearly badged **Cloud Verified** vs. **Offline Protocol Mode** state.
- **Voice agent**: speak a question (Sarvam speech-to-text with translation), let Gemini answer using tool calls (shelter lookup, safety manuals), and hear the reply as spoken English (Sarvam text-to-speech, `en-IN`).
- **Evacuation map and shelters** from PostGIS GeoJSON, with availability derived from `capacity - current_occupancy`, live volunteer tracking once dispatched, and turn-by-turn navigation with offline map regions.
- **Ground-truth reporting**: citizens and volunteers submit observed water depth to help validate predictions.

---

## 3. System Architecture

```mermaid
flowchart LR
    subgraph Sources["External data sources"]
        OM["Open-Meteo<br/>weather + flood API"]
        OWM["OpenWeatherMap<br/>(fallback)"]
        SLDC["KSEB / SLDC<br/>dam levels"]
        OSM["OpenStreetMap<br/>tiles / roads"]
    end

    subgraph Backend["FastAPI backend"]
        SCH["APScheduler<br/>hourly pipeline"]
        ML["ML engine<br/>XGBoost + SHAP"]
        CACHE[("In-memory<br/>live cache")]
        API["REST API + WebSocket"]
        AGT["Multi-agent workflow"]
        RAG["ChromaDB RAG<br/>NDMA / NOAA manuals"]
    end

    DB[("PostgreSQL + PostGIS")]
    GEM["Gemini"]
    SAR["Sarvam AI<br/>STT / TTS"]
    FCM["Firebase<br/>Cloud Messaging"]
    N8N["n8n / Twilio"]

    APP["Flutter app<br/>Android / iOS / Web"]

    OM --> SCH
    OWM -.-> SCH
    SLDC --> SCH
    SCH --> ML --> CACHE --> API
    API <--> DB
    API <--> AGT
    AGT --> GEM
    AGT --> RAG
    API --> RAG
    API --> GEM
    API --> SAR
    API --> FCM
    API --> N8N
    API <-->|REST + WS| APP
    OSM --> APP
    FCM --> APP
```

### Request flow: an SOS, end to end

1. The citizen taps SOS in the app. If the device is offline, the report is written to a local SQLite queue; otherwise `POST /api/reports` is called (or `POST /api/reports/lite` on 2G, or an SMS to the webhook with no data at all).
2. The backend stores the report as a PostGIS `POINT`, then runs the **allocation engine**: the first `available` volunteer within **5 km** is assigned, marked `busy`, and the ticket becomes `dispatched`.
3. A `new_sos_pending` event is broadcast over `/ws/dashboard` to command-centre clients.
4. Nearby users are asked to verify. Votes go to `POST /api/reports/{id}/verify`; at **3 confirmations** the ticket turns `verified`, a `sos_verified` event is broadcast, and the optional n8n webhook fires.
5. The assigned volunteer accepts (`en-route`), and the citizen can follow the volunteer's live location on the map.

### Hourly risk pipeline

```
fetch_open_meteo_data()          # 14 districts, 15-day rain + river discharge (OWM fallback on failure)
scrape_kseb_dam_levels()         # reservoir levels, safe defaults if the scrape fails
        │
        ▼
predict_district_risk()          # XGBoost -> class probabilities -> expected depth (cm)
        │
        ▼
kerala_live_cache                # served from RAM at /api/dashboard/live-kerala
        │
        └─ transition to "Severe"?  ->  pending AlertRecord  ->  responder approves  ->  `high_risk_alert` broadcast
```

---

## 4. Machine Learning & Inundation Methodology

### 4.1 Flood-risk classifier

| Item | Detail |
|---|---|
| Model | XGBoost multi-class booster (`app/ml_engine/flood_risk_model.json`) |
| Classes | `No Rain`, `Light`, `Moderate`, `Severe` |
| Features | 28 inputs in a fixed order, defined in `app/ml_engine/model_columns.json` |
| Feature sources | Open-Meteo (`om_*`), SRTM terrain (`srtm_*`), AWS gauges (`aws_*`), NWP (`nwp_*`), satellite (`sat_*`), radar (`rad_*`), plus `historical_flood_count`, cross-source `consensus_rainfall_*`, and seasonality (`season_sin`, `season_cos`, `is_monsoon_season`) |
| Decision rule | Predict **Severe** if `P(Severe) >= 0.20`; otherwise take the argmax class. A threshold well below 0.5 makes the system more sensitive to Severe events, which suits early warning |
| Risk score | Class mapped to a dial value: 10 / 35 / 65 / 90 |

**Robustness to missing inputs.** `predict_district_risk()` fills in whatever a live feed doesn't provide: rolling 3- and 7-day sums are derived from the 15-day total, missing AWS/satellite/NWP values are seeded from observed rainfall, and radar reflectivity falls back to a Marshall-Palmer estimate (`Z = 200·R^1.6`, `dBZ = 10·log10(Z)`) with small sensor-style jitter.

### 4.2 Expected inundation depth

Rather than a single hard class, depth is a probability-weighted expectation over class base depths calibrated against Sentinel-1 flood extents from the 2018 and 2019 Kerala floods:

```
estimated_depth_cm = Σ P(class) × base_depth(class)
base_depth(cm)     = { No Rain: 0.0, Light: 7.4, Moderate: 21.3, Severe: 51.3 }
```

A standalone terrain-based proxy lives in `app/ml_engine/inundation_depth_proxy.py` (`base_depth × clamp(1.5 − elevation/500 − slope/30, 0.3, 1.8)`) and is used by the offline output generator.

### 4.3 HAND-based raster overlay

`app/data/inundation_depth_overlay.png` (3137 × 5552 px, Web-Mercator-aligned) is a pre-rendered depth raster served at `GET /api/v1/map-overlay`, with its geographic bounds, legend and per-district summary at `GET /api/v1/map-bounds`.

- **Terrain model:** exponential decay in **HAND** (Height Above Nearest Drainage), with a fitted scale of about **1.63 m**, calibrated on Sentinel-1 flood maps from 2018 and 2019 (`app/data/terrain_calibration.json`).
- **River distance** was tested and dropped, because it added no signal beyond HAND.
- **Validation:** a pooled fit over roughly 3.6 million grid cells; cross-event, held-out within-district **AUC of about 0.83** (minimum 0.829 across the 2018/2019 held-out tests).
- **Legend:** under 10 cm, 10-20 cm, 20-50 cm, 50-100 cm, over 100 cm. Roads become impassable at **20 cm**.

> **Interpretation note.** Depth values are a *terrain-based proxy for relative severity*, not a calibrated hydrodynamic measurement. This is stated in the overlay metadata and should be stated in any downstream UI.

### 4.4 Explainability (SHAP)

- `app/data/global_feature_importance.json`: mean absolute SHAP per feature. Satellite-estimated rainfall, same-day rainfall and cross-source consensus rainfall rank highest.
- `app/data/district_explanations_latest.json`: top local drivers per district with signed contributions, served by `GET /api/v1/shap-drivers/{district}`.

### 4.5 Retrieval-augmented generation (RAG)

`seed_rag.py` chunks the manuals in `manuals/` (the NDMA Flood Management Guidelines and a NOAA/FEMA flood-safety guide) by paragraph (>100 characters) into a persistent **ChromaDB** collection named `survival_manuals`. The same store powers the chatbot, the voice agent's `query_safety_manuals` tool, and the Resource Allocator agent. A pre-built `chroma_db/` is committed, so re-seeding is optional.

---

## 5. Tech Stack

| Layer | Technologies |
|---|---|
| **Backend** | Python 3.12, FastAPI, Uvicorn, SQLAlchemy 2, GeoAlchemy2, Pydantic v2, APScheduler, httpx, BeautifulSoup |
| **Database** | PostgreSQL with PostGIS (spatial queries: `ST_DWithin`, `ST_X`, `ST_Y`) |
| **ML / geo** | XGBoost, SHAP, scikit-learn, pandas, NumPy, SciPy, GeoPandas, Shapely |
| **AI / RAG** | Google Gemini (`google-genai`), ChromaDB, Sarvam AI (speech-to-text and text-to-speech) |
| **Auth & messaging** | JWT (python-jose), bcrypt, Firebase Admin / FCM, Twilio-compatible SMS webhook, n8n webhook |
| **Frontend** | Flutter (Dart SDK `^3.12.1`), Riverpod, go_router, Dio, flutter_map, fl_chart |
| **On-device** | sqflite (offline queues and map tiles), flutter_secure_storage, geolocator, connectivity_plus, record / audioplayers / flutter_tts |
| **Infra** | Docker (python:3.12-slim), Render (backend), Vercel (Flutter web build) |

---

## 6. Repository Structure

```
PRAVAH/
├── app/                              # FastAPI backend
│   ├── main.py                       # App, scheduler, pipeline, WebSocket, core REST routes
│   ├── auth.py                       # JWT, bcrypt, role normalisation, RBAC dependencies
│   ├── models.py                     # SQLAlchemy/PostGIS models (User, Report, Shelter, AlertRecord, AgentRunLog, ...)
│   ├── schemas.py                    # Pydantic request/response schemas
│   ├── database.py                   # Engine + session (requires PostgreSQL/PostGIS)
│   ├── data_ingestion.py             # Open-Meteo / OpenWeatherMap / SLDC ingestion, four-pillar derivation
│   ├── agents.py, agent_manager.py   # Multi-agent Gemini workflow (+ RAG grounding)
│   ├── routers/
│   │   ├── inundation.py             # Time-lapse inundation simulation (T+0h .. T+3h)
│   │   ├── ps71.py                   # PS-71 heavy-rainfall prediction contract
│   │   ├── routing.py                # Blocked street bounding boxes for evacuation routing
│   │   ├── protocol.py               # NDMA protocol checklists by alert tier
│   │   ├── analytics.py              # Comparative storm telemetry
│   │   └── citizen_ops.py            # SMS broadcast fallback + ground-truth uploads
│   ├── ml_engine/
│   │   ├── service.py                # Inference, depth estimation, radar fallback
│   │   ├── road_hazards.py           # Flooded-road detection (GeoPandas)
│   │   ├── inundation_depth_proxy.py # Terrain-based depth proxy
│   │   ├── generate_inundation_output.py
│   │   ├── flood_risk_model.json     # Trained XGBoost model (4 classes)
│   │   └── model_columns.json        # Feature order, classes, severe threshold
│   └── data/                         # GeoJSON, HAND raster + bounds, SHAP + calibration JSON, grid parquet
├── frontend/                         # Flutter app (Android, iOS, Web, desktop scaffolds)
│   ├── lib/
│   │   ├── api/                      # PreciopsApi interface, Dio client, mock client, models
│   │   ├── core/                     # Router, theme, config (env.dart), roles, constants
│   │   ├── features/                 # auth, citizen, predictor, responder, volunteer, official,
│   │   │                             #   chat, voice, map, navigation, shelters, verification, alerts, sos, agent_hub
│   │   ├── providers/                # Riverpod providers
│   │   ├── services/                 # FCM, notifications, offline queues, SMS fallback, secure storage
│   │   └── widgets/                  # Shared UI components
│   ├── test/                         # Flutter tests
│   └── API_CONTRACT.md               # Original UI-to-API contract (see note in §9)
├── manuals/                          # Source documents for RAG (NDMA, NOAA/FEMA)
├── chroma_db/                        # Persisted ChromaDB vector store (pre-built)
├── ml_models/                        # Legacy XGBoost model + columns (12 features; superseded by app/ml_engine)
├── cache/                            # OpenStreetMap Overpass API JSON snapshots
├── seed_rag.py                       # Build/refresh the RAG vector store
├── build_offline_roads.py            # Build app/data/kerala_roads_major.geojson from an OSM shapefile
├── test_backend_integration.py       # Smoke test for /api/ml/predict-kerala
├── Dockerfile                        # Production image (honours $PORT)
├── prod_requirements.txt             # Pinned backend dependencies
└── features.docx                     # Original feature specification
```

> The root also contains archived snapshots (`floodops_backend_final.zip`, `backend_all.txt`, `full_backend_code.txt`) and a few empty placeholder folders. They are historical artifacts, not part of the running system.

**Naming note.** The project went through several working names. You will see **PRAVAH** (repository), **FloodOps** (API title, Dart models), and **Preciops/PrecipOps** (Flutter package and some strings) in the code. They all refer to this same system.

---

## 7. Getting Started

### 7.1 Prerequisites

- **Python 3.12**
- **PostgreSQL 14+ with the PostGIS extension** (required; the app refuses to start without a `postgresql://` URL)
- **Flutter SDK** matching Dart `^3.12.1` (for the app)
- *Optional:* Docker, a Google AI Studio key (Gemini), a Sarvam AI key, a Firebase service account, an OpenWeatherMap key

### 7.2 Backend

```bash
# 1. Clone
git clone https://github.com/saanvisarawat/PRAVAH.git
cd PRAVAH

# 2. Virtual environment + dependencies
python -m venv venv
source venv/bin/activate            # Windows: venv\Scripts\activate
pip install -r prod_requirements.txt

# 3. Database (once)
createdb pravah
psql -d pravah -c "CREATE EXTENSION IF NOT EXISTS postgis;"

# 4. Configuration: create a .env file in the repo root (see §8)
#    DATABASE_URL=postgresql://USER:PASSWORD@localhost:5432/pravah

# 5. Run from the repo root (ChromaDB and model paths are relative to it)
uvicorn app.main:app --reload --port 8000
```

On startup the app creates all tables, starts the scheduler and immediately kicks off the first pipeline run. Fetching 14 districts takes a little while (requests are paced to respect upstream rate limits), so `GET /api/dashboard/live-kerala` returns an "initializing" message until the first run completes.

Interactive API docs: **http://127.0.0.1:8000/docs**

#### Optional data preparation

```bash
# Re-seed the RAG vector store from ./manuals (a pre-built chroma_db/ is already included)
python seed_rag.py

# Build the flooded-road dataset (required for road-hazard output; the file is git-ignored)
#   1. Download the Kerala OSM roads shapefile (e.g. from Geofabrik) into ./kerala_roads/
#      so that kerala_roads/gis_osm_roads_free_1.shp exists.
#   2. Then:
python build_offline_roads.py       # writes app/data/kerala_roads_major.geojson
```

Without `kerala_roads_major.geojson`, the backend still runs, and road-hazard lists are simply empty.

#### Seed demo shelters (development only)

```bash
curl -X POST http://127.0.0.1:8000/api/dev/seed-shelters
```

This inserts five sample shelters. Note that the sample coordinates are in Delhi, so replace them with real Kerala shelter data for any realistic demo.

### 7.3 Docker

```bash
docker build -t pravah-backend .
docker run --rm -p 8000:8000 --env-file .env pravah-backend
```

The image listens on `$PORT` (default `8000`), so it runs unchanged on Render and similar platforms.

### 7.4 Frontend (Flutter)

```bash
cd frontend
flutter pub get
```

By default the app targets the hosted backend configured in `lib/core/config/env.dart`. Override at run time with `--dart-define`:

| Scenario | Command |
|---|---|
| **Web / desktop against local backend** | `flutter run -d chrome --dart-define=API_BASE_URL=http://127.0.0.1:8000 --dart-define=WS_BASE_URL=ws://127.0.0.1:8000` |
| **Android emulator** | `flutter run --dart-define=API_BASE_URL=http://10.0.2.2:8000 --dart-define=WS_BASE_URL=ws://10.0.2.2:8000` |
| **Physical device on Wi-Fi** | `flutter run --dart-define=API_BASE_URL=http://<PC_LAN_IP>:8000 --dart-define=WS_BASE_URL=ws://<PC_LAN_IP>:8000` |
| **Fully offline demo (mock data)** | `flutter run --dart-define=USE_MOCK_API=true` |

The backend URL can also be changed at run time from **Profile > Backend Server (Dev)**, and the choice is remembered across restarts.

**Push notifications.** Firebase options are included for the project's own Firebase app. To use your own, run `flutterfire configure` and replace `lib/firebase_options.dart` and `android/app/google-services.json`. If Firebase can't initialise, the app falls back to local notifications and a mock token so that nothing else breaks.

**Web release build:**

```bash
flutter build web --release
```

---

## 8. Configuration Reference

Create a `.env` file in the repository root (it is git-ignored).

| Variable | Required | Purpose |
|---|:---:|---|
| `DATABASE_URL` | **Yes** | PostgreSQL + PostGIS connection string (must start with `postgresql`) |
| `JWT_SECRET_KEY` | **Strongly recommended** | HS256 signing key. A hard-coded development fallback exists; **always override it in any shared or production environment** |
| `GOOGLE_API_KEY` | Optional | Enables Gemini for the chatbot, voice agent and multi-agent workflow. Without it, chat degrades to the offline protocol mode |
| `SARVAM_API_KEY` | Optional | Enables the voice agent (speech-to-text and text-to-speech) |
| `FIREBASE_CRED_PATH` | Optional | Path to a Firebase service-account JSON; required for sending masked-call FCM messages |
| `N8N_WEBHOOK_URL` | Optional | Webhook fired when a report reaches *verified* (e.g. to trigger a Twilio SMS workflow) |
| `OPENWEATHERMAP_API_KEY` | Optional | Fallback weather source when Open-Meteo is rate-limited or down |
| `PORT` | Optional | Port used by the Docker `CMD` (default `8000`) |

Example `.env`:

```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/pravah
JWT_SECRET_KEY=change-me-to-a-long-random-string
GOOGLE_API_KEY=...
SARVAM_API_KEY=...
FIREBASE_CRED_PATH=./firebase-service-account.json
N8N_WEBHOOK_URL=https://your-n8n-host/webhook/...
OPENWEATHERMAP_API_KEY=...
```

---

## 9. API Reference

Interactive documentation is generated by FastAPI at `/docs` (Swagger) and `/redoc`. Routes below are as implemented in `app/`.

> **Note on `frontend/API_CONTRACT.md`.** That file records the contract the UI was first built against and is partly out of date (for example, it names `/api/agents/analyze`, while the backend exposes `/api/agents/trigger`). Treat the running OpenAPI schema as the source of truth.

### Auth & users
| Method | Path | Description |
|---|---|---|
| `POST` | `/api/auth/register` | Create an account (`citizen` / `volunteer` / `official`) |
| `POST` | `/api/auth/login` | Returns `{access_token, token_type}` (JWT, 24 h) |
| `POST` | `/api/users/fcm-token` | Save the device's FCM token (no-op for guests) |

### SOS reports & verification
| Method | Path | Description |
|---|---|---|
| `POST` | `/api/reports` | Create an SOS (guest or authenticated); triggers allocation + broadcast |
| `POST` | `/api/reports/bulk` | Idempotent bulk upload of queued offline reports (dedupes on `client_timestamp` + user) |
| `POST` | `/api/reports/lite` | Query-string-only SOS for 2G/EDGE networks |
| `POST` | `/api/reports/sms-webhook` | Twilio/Exotel webhook; body format `SOS <lat> <lng> <message>`; replies with TwiML |
| `GET` | `/api/reports` | Verified reports as plain JSON with lat/lng |
| `POST` | `/api/reports/{ticket_id}/verify` | Vote `{is_verified: bool}`; 3 confirmations marks the report *verified* |
| `GET` | `/api/reports/{ticket_id}/volunteer-location` | Live location of the volunteer assigned to a ticket |

### Volunteers & officials
| Method | Path | Description |
|---|---|---|
| `POST` | `/api/volunteers/location` | Update status (`available`/`busy`/`offline`), skills and GPS fix |
| `GET` | `/api/volunteers/tasks` | Tasks assigned to the current volunteer |
| `POST` | `/api/volunteers/tasks/{task_id}/accept` | Accept a task, set `en-route`, notify the citizen |
| `GET` | `/api/officials/reports` | All reports, any status *(official only)* |
| `GET` | `/api/officials/volunteers` | All volunteers with last known position *(official only)* |
| `POST` | `/api/officials/reports/{report_id}/assign` | Manual dispatch / reassignment *(official only)* |
| `POST` | `/api/emergency/trigger-masked-call` | Send an `EMERGENCY_INCOMING_CALL` FCM data message to a volunteer |

### Alerts (human-in-the-loop)
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/alerts/pending` | Model-raised alerts awaiting review *(volunteer or official)* |
| `POST` | `/api/alerts/{id}/approve` | Approve and broadcast `high_risk_alert` |
| `POST` | `/api/alerts/{id}/reject` | Reject a pending alert |

### Prediction, telemetry & maps
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/dashboard/live-kerala` | Cached per-district predictions, four-pillar telemetry, depth, reservoirs |
| `POST` | `/api/ml/predict-kerala` | Risk, depth, telemetry and flooded roads for `{lat, lon, district}` |
| `POST` | `/api/predict/risk` | Direct model inference from raw feature inputs |
| `GET` | `/api/v1/map-overlay` | HAND inundation raster (PNG) |
| `GET` | `/api/v1/map-bounds` | Overlay bounds, legend and per-district summary |
| `GET` | `/api/v1/shap-drivers/{district}` | Local SHAP drivers for a district |
| `GET` | `/api/v1/roads/hazards/{district}` | Roads exceeding the 20 cm flood threshold |
| `GET` | `/api/shelters/geojson` | Shelters as a GeoJSON FeatureCollection |
| `GET` | `/api/v1/inundation/{predict,simulate}` | Time-lapse inundation frames (`district`, `scenario`) |
| `GET` | `/api/v1/heavy-rainfall/predict` | Heavy-rainfall prediction contract (PS-71 shaped response) |
| `GET` | `/api/v1/routing/blocked-nodes` | Impassable bounding boxes for evacuation routing |
| `GET` | `/api/v1/analytics/storm-comparison` | Current storm vs. historical benchmarks |

### AI, protocols & citizen operations
| Method | Path | Description |
|---|---|---|
| `POST` | `/api/chat` | RAG chatbot (`message`, `language`); returns `mode: online \| offline` |
| `POST` | `/api/voice/agent` | Multipart `lat`, `lng`, `audio_file`; returns a spoken WAV answer |
| `POST` | `/api/agents/trigger` | Run the four-agent workflow for `{district, incident_data}` |
| `GET` | `/api/v1/agents/protocol` | NDMA checklist by `alert_level`, `radar_dbz`, `satellite_temp_k` |
| `POST` | `/api/v1/citizen/broadcast` | Broadcast an advisory to connected clients |
| `POST` | `/api/v1/citizen/verification/upload` | Submit a ground-truth water-depth observation |

### Utility
| Method | Path | Description |
|---|---|---|
| `GET` | `/`, `/health` | Liveness checks |
| `POST` | `/api/dev/seed-shelters` | Development-only sample data |

---

## 10. Real-Time Events (WebSocket)

Connect to `ws(s)://<host>/ws/dashboard`. The server pushes one JSON object per frame; clients don't need to send anything.

| `type` | Emitted when | Key fields |
|---|---|---|
| `new_sos_pending` | A new SOS is created (any channel) | `ticket_id`, `latitude`, `longitude`, `description`, `assigned_volunteer_id` |
| `sos_verified` | A ticket reaches 3 confirmations | `ticket_id`, `description`, `status` |
| `volunteer_assigned` | An official manually dispatches a volunteer | `ticket_id`, `assigned_volunteer_id`, `assigned_volunteer_name` |
| `volunteer_en_route` | A volunteer accepts a task | `ticket_id`, `volunteer_id`, `volunteer_name` |
| `high_risk_alert` | A responder approves a model-raised alert | `district`, `risk_score`, `alert_level`, `timestamp` |
| `advisory_broadcast` | An advisory is broadcast to a district/zone | `district`, `zone_id`, `alert_message` |
| `ground_truth_submitted` | A ground-truth observation arrives | `district`, `latitude`, `longitude`, `observed_water_depth_meters` |

---

## 11. Roles & Access Control

The database stores three roles; the app presents them as the three tiers defined by the problem statement.

| Stored role (`UserRole`) | Tier | Typical user | Lands on |
|---|---|---|---|
| `official` | **PREDICTOR** | IMD meteorologists, MoES officials | Predictor dashboard (telemetry, inundation simulation, SHAP, overlay) |
| `volunteer` | **RESPONDER** | District authorities, NDMA/NDRF, field volunteers | Responder dashboard (IMD advisory, tactical map, NDMA checklist, storm analytics, SOS queue, volunteer hub) |
| `citizen` | **CITIZEN** | General public | Citizen shell: Home, Hub, Map, Ragbot, Verify |

- Authentication uses **JWT bearer tokens** (HS256, 24-hour expiry) with **bcrypt**-hashed passwords.
- `auth.py` provides `require_predictor` and `require_responder` dependencies and a role-normalisation layer that maps legacy names onto the tiers. The `/api/officials/*` routes enforce `official`, and alert review accepts `volunteer` or `official`.
- Several citizen-facing endpoints (SOS creation, verification votes, chat) intentionally allow guest access, so people in an emergency are never blocked behind a login.

---

## 12. Offline & Low-Connectivity Design

Flood zones lose connectivity first. PRAVAH degrades through several layers rather than failing:

| Layer | Mechanism |
|---|---|
| **Offline queue** | SOS reports are written to local SQLite with a client-generated ID and timestamp, then drained through `POST /api/reports/bulk` when connectivity returns. The backend deduplicates. |
| **Lite SOS** | `POST /api/reports/lite` takes everything in the query string. The client persists each submission and retries every 8 s until a `200 OK`, including across app restarts. |
| **SMS fallback** | With zero data, the app opens the native SMS composer pre-filled with `SOS <lat> <lng> <description>`, addressed to the Twilio-backed webhook. |
| **Offline chatbot** | If Gemini is unavailable, `/api/chat` returns the retrieved NDMA protocol passages directly, labelled *Offline Protocol Mode*. With no match at all it returns a safe default ("move to high ground"). |
| **Offline maps** | OSM raster tiles for chosen regions are downloaded and cached on-device (zoom 10-14) with de-duplicated storage. |
| **Resilient ingestion** | Open-Meteo failures fall back to OpenWeatherMap; dam scraping falls back to default values; the radar pillar falls back to a synthesized estimate. |
| **Mock mode** | `USE_MOCK_API=true` runs the entire UI with realistic Kerala mock data and no backend. |

---

## 13. Deployment

**Backend (Render or any container host).** The included `Dockerfile` builds from `python:3.12-slim`, installs `libpq-dev` and `gcc` for `psycopg2`, installs `prod_requirements.txt`, and starts Uvicorn on `$PORT`. Provision a managed PostgreSQL instance with PostGIS enabled, set the environment variables from §8, and deploy.

**Frontend.** Build with `flutter build web --release` and host the `build/web` output on any static host (the project's web build was served through Vercel). Android and iOS targets use the standard Flutter toolchain. Point the build at your backend with `--dart-define=API_BASE_URL=...` and `--dart-define=WS_BASE_URL=...`.

---

## 14. Testing

```bash
# Backend smoke test (server must be running and the first pipeline run complete)
python test_backend_integration.py

# Flutter unit and widget tests
cd frontend && flutter test
```

`test_backend_integration.py` posts `{lat: 9.9189, lon: 77.1025, district: "Idukki"}` to `/api/ml/predict-kerala` and prints the response. The Flutter tests currently cover Kerala telemetry/chat models and a widget smoke test.

---

## 15. Current Limitations & Simulated Components

Transparency matters in safety software. These parts of the system are **simulated, partial or hard-coded** today:

- **Static district features.** Elevation, slope, river distance, historical flood counts and rolling-rain baselines for each district are hard-coded in the pipeline; only rainfall and river discharge are truly live.
- **Synthesized pillars.** Radar reflectivity, cloud-top temperature and parts of the NWP/AWS/satellite inputs are *derived* from Open-Meteo values via empirical relations; they are not direct IMD, MOSDAC or INSAT-3DR feeds. Missing model inputs are defaulted as described in §4.1.
- **Placeholder "top factors".** The live cache reports a fixed driver list; true per-district drivers come from the precomputed SHAP files instead.
- **Demo data in several routers.** `ps71`, `routing` (blocked bounding boxes), `protocol` (checklists), `analytics` (historical benchmarks), and the scenario values in `inundation` are scripted and deterministic. The SMS "recipient count" returned by `/citizen/broadcast` is simulated.
- **Satellite SAR job.** `fetch_satellite_sar_data` is a scheduled stub that logs only.
- **Push delivery.** Volunteer-assignment and nearby-verification pushes are currently logged server-side; masked-call FCM messages *are* sent when Firebase credentials are configured.
- **Shelter lookup in the voice agent.** The `find_nearest_shelter` tool returns a fixed example shelter.
- **Raster freshness.** The overlay is a pre-rendered snapshot (see `as_of_date` in `map-bounds`) and is not regenerated by the live pipeline.
- **Model IDs.** The chatbot and voice agent request `gemini-1.5-flash` while the multi-agent workflow requests `gemini-3.5-flash`. Check these against the models currently available to your API key.
- **Validation scope.** Held-out metrics come from the 2018 and 2019 events used for calibration; the system has not been validated operationally.

---

## 16. Security & Hardening Checklist

Before any non-demo deployment:

- [ ] Set a strong `JWT_SECRET_KEY` (remove reliance on the built-in fallback).
- [ ] Restrict CORS (`allow_origins=["*"]` is set for development).
- [ ] Protect or remove `POST /api/dev/seed-shelters`.
- [ ] Add authentication and rate limiting to open mutation endpoints (e.g. `/api/agents/trigger`, `/api/v1/citizen/broadcast`, report verification) and add per-user vote de-duplication.
- [ ] Enable TLS verification in the dam scraper (it currently disables certificate checks for the SLDC site).
- [ ] Restrict the Firebase/Google API keys in the committed client config to your app identifiers, or rotate and regenerate your own.
- [ ] Validate and bound inputs to the SMS webhook; verify Twilio/Exotel request signatures.
- [ ] Keep `.env` and service-account JSON out of version control (already git-ignored).
- [ ] Add database migrations (Alembic) in place of `create_all`.

---

## 17. Roadmap

- Replace synthesized pillars with direct **IMD AWS / DWR**, **MOSDAC INSAT-3DR**, and **NCMRWF** feeds.
- Live per-district feature engineering (rolling windows, soil moisture, reservoir release schedules) instead of static baselines.
- Regenerate the HAND inundation raster dynamically from each forecast, including physics-informed validation.
- Proper offline road-graph routing (the app currently caches tiles but routes online via OSRM, with a straight-line fallback).
- Real push delivery for volunteer assignment and community verification; delivery receipts.
- Automated test suite and CI, plus Alembic migrations.
- Extend beyond Kerala using the pan-India city registry already present in the inundation router.

---

## 18. Contributing

Contributions are welcome.

1. Fork the repository and create a feature branch: `git checkout -b feature/your-change`
2. Keep backend changes consistent with the existing Pydantic schemas and router layout, and keep Flutter screens talking only to the `PreciopsApi` interface (never directly to the mock or Dio clients).
3. Run `flutter analyze` and `flutter test` for frontend changes.
4. Open a pull request describing the motivation, the change, and how you verified it.

Please open an issue first for larger changes.

---

## 19. License

No license file is currently included.

---

## 20. Acknowledgements

- **IMD, MoES, ISRO/MOSDAC and NCMRWF**, whose problem statement and data ecosystem motivated the design.
- **NDMA** (Flood Management Guidelines) and **NOAA/FEMA** (flood safety guidance) for the RAG corpus.
- **Open-Meteo**, **OpenWeatherMap**, **KSEB/SLDC Kerala**, **OpenStreetMap contributors** (ODbL), and **Copernicus Sentinel-1** for data used in ingestion and calibration.
- **Google Gemini**, **Sarvam AI**, **Firebase**, **ChromaDB**, **XGBoost**, **SHAP**, **FastAPI**, **PostGIS** and **Flutter** for the tooling that makes this possible.
