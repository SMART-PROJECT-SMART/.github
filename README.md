# SMART - Simulation and Management for Advanced Reconnaissance and Tactical operations

## Overview

SMART is a comprehensive simulation system designed for advanced reconnaissance and tactical operations. The project consists of multiple components including simulation engines, telemetry devices, and various supporting services that work together to provide a complete tactical simulation environment.

## System Architecture


<img width="11558" height="7194" alt="FULL" src="https://github.com/user-attachments/assets/3b2a76ac-3e72-4eb6-9b1b-a35954d7ee66" />


*System Architecture Design - Visual representation of the SMART system components and their interactions*

## Architecture

The SMART system is built with a microservices architecture consisting of:

### Core Components

1. **Simulation Service** - Main simulation engine handling UAV operations, flight paths, and tactical scenarios
2. **TelemetryDevice Service** - Packet sniffing and telemetry data collection service
3. **Docker Infrastructure** - Containerized messaging and gateway services

### Key Features

- **UAV Simulation**: Support for both armed and surveillance UAVs with realistic flight path calculations
- **Real-time Telemetry**: Live data collection and processing from simulated devices
- **Communication Systems**: Integrated Communication Controller for multi-channel operations
- **ICD Integration**: Support for North and South ICD (Interface Control Document) configurations
- **Flight Path Management**: Advanced motion calculation and orientation control
- **Messaging Infrastructure**: Kafka-based messaging system for inter-service communication
- **API Gateway**: Nginx-based gateway for service orchestration

## Project Structure

```
├── Simulation/                 # Main simulation service
│   ├── Controllers/           # API controllers
│   ├── Models/               # Data models and entities
│   ├── Services/             # Business logic services
│   ├── Common/               # Shared constants and enums
│   └── Configuration/        # Service configuration
├── TelemetryDevice/          # Telemetry collection service
├── docker/                   # Docker infrastructure
│   ├── docker-compose.messaging.yml
│   └── docker-compose.gateway.yml
└── scripts/                  # Automation scripts
```

## Getting Started

### Prerequisites

- .NET 9.0 or later
- Docker and Docker Compose
- PowerShell (for Windows automation scripts)

### Quick Start

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd SMART
   ```

2. **Start the infrastructure services**

   ```bash
   # Start all Docker services
   .\scripts\compose-all.bat

   # Or start individual services
   .\scripts\compose-messaging.bat    # Kafka messaging
   .\scripts\compose-gateway.bat      # Nginx gateway
   ```

3. **Run the applications**
   ```bash
   # Run all applications
   .\run.bat
   ```

### Individual Service Management

#### Simulation Service

```bash
cd Simulation\Simulation
dotnet run
```

#### TelemetryDevice Service

```bash
cd TelemetryDevice\TelemetryDevice
dotnet run
```

## Configuration

### ICD Settings

The system supports configurable ICD (Interface Control Document) settings for North and South configurations, located in:

- `Simulation/Files/ICD/North_ICD.json`
- `Simulation/Files/ICD/South_ICD.json`

### Communication Channels

Multi-channel communication is supported through the Communication Controller with configurable port switching and channel management.

### UAV Properties

Configurable UAV properties including:

- Sensor types and capabilities
- Weapon system configurations
- Flight characteristics
- Mission context parameters

## API Endpoints

### Simulation Controller

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

### Messaging Stack (Kafka)

```yaml
# Kafka configuration for inter-service messaging
services:
  kafka:
    image: confluentinc/cp-kafka:7.7.0
    ports:
      - "9092:9092"
      - "9093:9093"
```

### Gateway Stack (Nginx)

```yaml
# API Gateway for service routing
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:8080"
```

## Development

### Building the Project

```bash
# Build all projects
dotnet build

# Build specific project
dotnet build Simulation/Simulation.sln
```

### Running Tests

```bash
# Run all tests
dotnet test

# Run tests for specific project
dotnet test Simulation/
```

### Code Structure

The project follows Clean Architecture principles with:

- **Controllers**: API endpoints and request handling
- **Services**: Business logic and domain operations
- **Models**: Data entities and DTOs
- **Common**: Shared utilities and constants
- **Configuration**: Service configuration and settings

## Monitoring and Telemetry

The system includes comprehensive telemetry collection through:

- Packet sniffing capabilities
- Real-time performance metrics
- Communication channel monitoring
- UAV operational status tracking

## Contributing

Please follow the established coding standards and ensure all tests pass before submitting pull requests.

## License

[License information to be added]

## Support

For technical support and questions, please refer to the project documentation or contact the development team.
