# SMART - Simulation and Management for Advanced Reconnaissance and Tactical operations

## Overview

SMART is a comprehensive simulation and command-support stack for advanced reconnaissance and tactical operations. It combines **real-time UAV simulation**, **live and historical telemetry**, **mission and assignment planning**, **device and sleeve coordination**, **map-backed situational displays**, and **MongoDB-backed records** behind an **Angular** operator client. Services communicate over **HTTP APIs**, **Kafka**, and **MongoDB**, with **Nginx** as a gateway and **Docker Compose** for local infrastructure.

Operational details (ports, credentials, and startup variants) are maintained in **`scripts/README.md`** alongside compose helpers under `scripts/` and compose definitions under `docker/`.

## System Architecture

<img width="11558" height="7194" alt="FULL" src="https://github.com/user-attachments/assets/3b2a76ac-3e72-4eb6-9b1b-a35954d7ee66" />

_System Architecture Design - Visual representation of the SMART system components and their interactions_

## Architecture

The system is organized as **multiple ASP.NET Core services** plus an **Angular** front end, supported by **containerized infrastructure** (MongoDB, Kafka, Nginx). Typical local flows start **Docker** first, then **.NET** services (often with **DeviceManager** early because other components depend on it), then the **client** dev server.

## Core backend services

| Component            | Role                                                                                                                                                                                                      |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Simulation**       | Main simulation engine: UAV flight physics, flight paths, motion and orientation, Quartz-driven jobs, multi-UAV scenarios, REST control, **ICD-aligned** telemetry semantics. See `Simulation/README.md`. |
| **TelemetryDevice**  | Live telemetry path: packet capture, validation, ICD decoding, pipeline-based processing, dynamic ports and multi-device handling. See `TelemetryDevice/README.md`.                                       |
| **DeviceManager**    | Central coordination: **MongoDB** persistence, **Kafka**, and notification-style integration toward Simulation, TelemetryDevice, ACM, and MongoConsumer so the fleet state stays coherent.                |
| **ACM**              | Airborne / assignment-side coordination: **sleeve** workflows, **DeviceManager** and **Simulator** HTTP clients, **Kafka**, assignment services, and **UAV status consumption**.                          |
| **Mission Service**  | Mission domain: **assignments** (including algorithm hooks), **assignment results**, **mission status**, **mission execution**, UAV services, **MongoDB**, and background processing.                     |
| **LTS**              | Live telemetry support APIs: **UAV telemetry data** and **wanted-fields** configuration used by downstream consumers.                                                                                     |
| **TelemetryRecords** | Durable telemetry and mission-side records over **MongoDB**: telemetry data APIs and assignment-related endpoints.                                                                                        |
| **MongoCon sumer**   | Stream and integration worker: **Kafka**, **HTTP clients**, **MongoDB**, and **UAV change handlers** to propagate updates into persistence and other workflows.                                           |
| **MapServer**        | Map tile and map services used by the gateway and UI; tiles are mounted into **Nginx** for static map delivery.                                                                                           |

## Angular client (`client/`)

The **Angular** application is the primary operator interface. Major areas include:

- **Assignment** — mission and scenario selection, assignment m
  anagement, **map-centric review** (UAV and mission markers, filters, tooltips, algorithm panels), summaries, and related dialogs.
- **Archive** — historical mission telemetry views, time-range selection, and comparison/diff style tooling where implemented.
- **Routing and modules** — standard SPA navigation across these domains with service calls into the APIs above.

Use `client/README.md` for CLI versions, `ng serve`, builds, and tests.

## Data platform and infrastructure

| Layer                                                                              | Purpose                                                                                                                |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **MongoDB** (`docker/docker-compose.mongodb.yml`)                                  | Primary document store for missions, telemetry records, device/manager state, and related services.                    |
| **Kafka** (`docker/docker-compose.messaging.yml`)                                  | Inter-service events and streaming; consumed by MongoConsumer, ACM, DeviceManager integrations, and related producers. |
| **Nginx gateway** (`docker/docker-compose.gateway.yml`, `docker/nginx/nginx.conf`) | HTTP entry, routing, and **map tile** hosting from MapServer content.                                                  |
| **Core** (`Core/`)                                                                 | Shared **ICD** JSON and shared libraries referenced by simulation and telemetry stacks.                                |
| **Shared** (`Shared/`)                                                             | Additional shared assets or libraries as used by solutions in this repository.                                         |

## Key features (system-wide)

**Simulation and UAV modeling**

- Armed and surveillance **UAV types** with distinct mass, speed, altitude, sensors, and weapons where modeled (see Simulation README for platform list).
- **Flight path** services: motion, orientation, speed control, and validation.
- **Quartz.NET** scheduling for periodic telemetry, simulation ticks, and maintenance-style jobs.
- **ICD**-driven payloads for North/South-style configurations (files under `Core/Core/Files/ICD/`).

**Telemetry and sensing**

- **Live capture** and decode pipelines (checksums, ICD decoders, outputs).
- **Multi-port** and **port manager** behavior for channel switching.
- **TelemetryRecords** and **LTS** for serving and shaping telemetry for UI and analytics.

**Missions, assignments, and C2-style workflows**

- **Mission Service** APIs for assignments, results, and mission status plus execution pipeline.
- **ACM** integration with assignments, Kafka, and live UAV status.
- **Client** workflows for assignment review on maps, scenario handling, and mission summaries.

**Integration and operations**

- **DeviceManager** as the hub for cross-service notifications and persistence alignment.
- **MongoConsumer** for Kafka-driven persistence and UAV change handling.
- **MapServer** plus gateway for **map-backed** situational awareness.
- **Scripted** Docker and multi-service startup documented under `scripts/README.md`.

## Project structure

```
├── client/                     # Angular SPA (assignment, archive, mission telemetry UI)
├── Simulation/                 # UAV flight simulation service
├── TelemetryDevice/            # Live telemetry capture and pipeline
├── DeviceManager/              # Coordination hub (MongoDB, Kafka, notifications)
├── ACM/                        # Assignment / sleeve coordination, Kafka, clients
├── Mission Service/            # Missions, assignments, execution, MongoDB
├── LTS/                        # Live telemetry and wanted-fields APIs
├── TelemetryRecords/         # MongoDB-backed telemetry and assignment records
├── MongoConsumer/            # Kafka consumers, MongoDB, UAV change handlers
├── MapServer/                  # Map services and tiles for the gateway
├── Core/                       # ICD definitions and shared code
├── Shared/                     # Additional shared libraries or assets
├── docker/                     # docker-compose.*.yml (MongoDB, Kafka, gateway)
├── scripts/                    # compose-*.bat, run-all.bat, UML helpers, utilities
└── UML/                        # Generated or maintained UML material
```

## Getting Started

### Prerequisites

- .NET 9.0 or later (match the SDK used by your solution files)
- Docker and Docker Compose
- PowerShell or Command Prompt (for Windows automation scripts)
- Node.js and npm (for `client/` when using `ng serve` or `ng build`)

### Quick Start

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd SMART
   ```

2. **Start the infrastructure services**

   From the repository root on Windows:

   ```bat
   scripts\compose-all.bat
   ```

   Or start individual stacks via the wrappers in `scripts/`:

   ```bat
   scripts\compose-messeging.bat
   scripts\compose-gateway.bat
   ```

   The Compose files live under `docker/`; the batch files change into that directory and run `docker compose`.

3. **Run the applications**

   ```bat
   scripts\run-all.bat
   ```

   For Docker plus applications in one flow, see `scripts\run-and-compose-all.bat`. For full-stack startup order, ports, and troubleshooting, see **`scripts/README.md`**.

### Individual Service Management

#### Simulation Service

```bash
cd Simulation/Simulation
dotnet run
```

#### TelemetryDevice Service

```bash
cd TelemetryDevice/TelemetryDevice
dotnet run
```

#### DeviceManager

```bash
cd DeviceManager/DeviceManager
dotnet run
```

#### ACM

```bash
cd ACM/ACM
dotnet run
```

#### Mission Service

```bash
cd "Mission Service/Mission Service"
dotnet run
```

#### LTS

```bash
cd LTS/LTS
dotnet run
```

#### TelemetryRecords

```bash
cd TelemetryRecords/TelemetryRecords
dotnet run
```

#### MongoConsumer

```bash
cd MongoConsumer/MongoConsumer
dotnet run
```

#### MapServer

```bash
cd MapServer/MapServer
dotnet run
```

#### Angular client

```bash
cd client
npm install
ng serve
```

## Configuration

### ICD Settings

ICD-related JSON used by configuration is under **Core**:

- `Core/Core/Files/ICD/NorthICD_telemetry.json`
- `Core/Core/Files/ICD/SouthICD_telemetry.json`

`Simulation/Simulation/appsettings.json` references ICD paths via configuration (verify `ICD` section for the active path on your machine).

### Communication Channels

Multi-channel communication is supported through the Communication Controller with configurable port switching and channel management.

### UAV Properties

Configurable UAV properties including:

- Sensor types and capabilities
- Weapon system configurations
- Flight characteristics
- Mission context parameters

## API Endpoints

### Simulation (representative)

- Flight path simulation
- Destination switching
- Mission management

### Communication Controller

- Port switching operations
- Channel management
- Communication protocols

### Telemetry Sniffers Controller

- Packet capture operations
- Telemetry data retrieval
- Device monitoring

### Other services (representative routes)

- **Mission Service** — `api/Assignment`, `api/AssignmentResult`, `api/mission-status`, and related controllers under `Mission Service/Mission Service/Controllers/`.
- **LTS** — `api/UAVTelemetryData`, `api/wanted-fields`.
- **TelemetryRecords** — `api/TelemetryData`, `api/Assignment`.
- **MapServer** — health and map APIs under `MapServer/MapServer/Controllers/` (see gateway routing in `docker/nginx/nginx.conf`).

Use each service’s **Swagger** URL when running locally (typically under `/swagger` where enabled).

## Services Architecture

### Flight Path Service

- **Motion Calculator**: Real-time position and velocity calculations
- **Orientation Calculator**: Heading and attitude management
- **Speed Controller**: Velocity profile management
- **Validators**: Input validation and constraint checking

### UAV Manager

Manages different UAV types:

- Armed UAVs with weapon systems
- Surveillance UAVs with sensor packages
- Mission context and operational parameters

### Port Manager

Handles communication port allocation and management for multi-channel operations.

### Quartz Scheduler

Background job scheduling for:

- Periodic telemetry collection
- Simulation state updates
- System maintenance tasks

## Docker Infrastructure

### Messaging stack (Kafka)

Defined in `docker/docker-compose.messaging.yml` (Kafka 7.7.0-style configuration; see file for full environment and volumes).

### Gateway stack (Nginx)

Defined in `docker/docker-compose.gateway.yml`. Host port **80** is mapped to port **8080** in the container; map tiles are mounted from `MapServer`. See that file and `docker/nginx/nginx.conf` for routing details.

### MongoDB

Defined in `docker/docker-compose.mongodb.yml` for local document storage used by multiple services.

## Development

### Building the project

```bash
dotnet build Simulation/Simulation.sln
```

Build a specific `.csproj` when you change one service only (examples in repository root `CLAUDE.md`).

### Running tests

```bash
dotnet test Simulation/
```

Adjust the path for other test projects as needed.

### Code structure

Simulation-oriented services commonly use:

- **Controllers**: API endpoints and request handling
- **Services**: Business logic and domain operations
- **Models**: Data entities and DTOs
- **Common**: Shared utilities and constants
- **Configuration**: Service configuration and settings

Other folders may differ; follow the layout of the project you edit.

## Monitoring and telemetry

The system includes telemetry collection through packet sniffing, performance metrics, communication channel monitoring, and UAV operational status tracking (see TelemetryDevice, LTS, TelemetryRecords, and MongoConsumer).

## Contributing

Follow the established coding standards and ensure tests pass before submitting pull requests. Cross-service C# conventions are summarized in `CLAUDE.md` at the repository root.

## License

[License information to be added]

## Support

For technical support and questions, refer to project documentation under each component, `scripts/README.md`, and `client/README.md` where applicable.
