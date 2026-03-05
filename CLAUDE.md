# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Event-Driven Vehicle Intake system built using **Axon Framework** for CQRS and Event Sourcing patterns. The project is a multi-module Maven build that provides API definitions (commands, events, and queries) for two bounded contexts: vehicles and lots.

## Architecture

### Multi-Module Structure

The project consists of two API modules that define shared contracts:

- **vehicleapi**: Defines commands, events, and queries for vehicle operations (purchase, sell, send to lot)
- **lotapi**: Defines commands, events, and queries for lot operations (create, update)

Both modules are:
- Published as independent artifacts to GitHub Packages
- Pure API definitions (no service implementations)
- Designed to be consumed by separate microservices (query and update services for each domain)

### Axon Framework Integration

This project uses **Axon Framework 4.10.3** for CQRS/Event Sourcing:

- Commands use `@TargetAggregateIdentifier` annotation to route to aggregates
- Events represent domain changes (e.g., `VehiclePurchaseEvent`, `LotCreateEvent`)
- Queries use filter objects for flexible querying (e.g., `VehicleSummaryFilter`)
- All DTOs use Lombok for boilerplate reduction (`@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`)

### Technology Stack

- Java 17
- Spring Boot 3.5.0 (parent POM)
- Axon Framework 4.10.3
- Lombok 1.18.34
- Micrometer 1.14.2 (observability)
- Jakarta Persistence API (for entity annotations)
- Maven for build management

## Build Commands

### Build the entire project
```bash
mvn clean compile
```

### Run tests
```bash
mvn test
```

### Package artifacts (skip tests)
```bash
mvn package -DskipTests
```

### Build and install locally
```bash
mvn clean install
```

### Build specific module
```bash
mvn clean compile -pl vehicleapi
mvn clean compile -pl lotapi
```

### Deploy to GitHub Packages
```bash
# Requires GITHUB_TOKEN environment variable
mvn deploy -pl vehicleapi,lotapi -am
```

## Development Workflow

### Adding New Commands or Events

When adding new commands or events to either API module:

1. Create the command/event class in the appropriate package:
   - Vehicle domain: `net.kamradtfamily.vehicleevent.api`
   - Lot domain: `net.kamradtfamily.vehicleevent.lot.api`

2. Use Lombok annotations for standard patterns:
   ```java
   @Data
   @Builder
   @NoArgsConstructor
   @AllArgsConstructor
   public class NewCommand {
       @TargetAggregateIdentifier  // Required for commands
       String id;
       // other fields
   }
   ```

3. Events should mirror command structure but represent past tense
4. Commands target aggregates; events are facts that happened
5. Rebuild and redeploy both modules when API contracts change

### Module Dependencies

- Both API modules depend on `axon-modelling` for annotations
- Lombok is configured as an annotation processor in the compiler plugin
- Jakarta Persistence API is available for entity-style annotations

## CI/CD

The project uses GitHub Actions:

- **ci.yml**: Runs on push/PR to main - builds, tests, and packages artifacts
- **publish-packages.yml**: Publishes vehicleapi and lotapi to GitHub Packages on release or manual trigger
- **pr-checks.yml**: Additional PR validation
- **codeql.yml**: Security scanning

## Package Distribution

Both API modules are configured to publish to GitHub Packages:
- Repository: `https://maven.pkg.github.com/rkamradt/vehicleevent`
- Group ID: `net.kamradtfamily`
- Artifacts: `vehicleapi` and `lotapi`

These artifacts are meant to be consumed by separate microservices implementing the query and update (command) sides of the CQRS pattern.

## Domain Model

### Vehicle Domain

**Commands:**
- `VehiclePurchaseCommand`: Purchase a vehicle (id, price, type)
- `VehicleSellCommand`: Sell a vehicle (id, price, type, saleDate)
- `VehicleSendToLotCommand`: Send vehicle to a lot (id, lotId, lotName, type)

**Events:**
- `VehiclePurchaseEvent`: Vehicle purchased
- `VehicleSellEvent`: Vehicle sold (includes saleDate, buyer info)
- `VehicleSendToLotEvent`: Vehicle sent to lot

**Queries:**
- `FetchVehicleSummaryQuery`: Retrieve vehicle summaries with optional filters
- `VehicleSummary`: Read model representation
- `VehicleSummaryFilter`: Filter criteria for queries

### Lot Domain

**Commands:**
- `LotCreateCommand`: Create a new lot (id, name, manager)
- `LotUpdateCommand`: Update lot details (id, name, manager)

**Events:**
- `LotCreateEvent`: Lot created
- `LotUpdateEvent`: Lot updated

**Queries:**
- `FetchLotSummaryQuery`: Retrieve lot summaries with filters
- `FetchLotByNameQuery`: Find lot by name
- `LotSummary`: Read model representation
- `LotSummaryFilter` / `LotByNameFilter`: Filter criteria

## Important Notes

- This repository contains only API definitions, not implementations
- Actual aggregate, saga, and projection implementations live in separate services
- Changes to these APIs are breaking changes for downstream services
- Version bumps should follow semantic versioning due to contract nature
