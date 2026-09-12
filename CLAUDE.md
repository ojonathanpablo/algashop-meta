# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AlgaShop Meta is a microservices architecture project that uses **Git submodules** to organize independent services. This meta-repository coordinates multiple microservices, each maintained in separate repositories.

Current microservices:
- **Ordering Service** (`microservices/ordering/algashop-ordering/`) - Customer and order management

## Architecture

### Git Submodules Structure

This repository uses Git submodules for microservice management:
- Each microservice is an independent Git repository
- Services are referenced in `.gitmodules`
- Services can be developed, versioned, and deployed independently

When cloning or updating:
```bash
# Clone with submodules
git clone --recursive <repo-url>

# Update submodules after pulling
git submodule update --init --recursive

# Pull latest changes in submodules
git submodule update --remote
```

### Microservice Technology Stack

**Ordering Service:**
- Java 21 (required)
- Spring Boot 3.5.9
- Gradle 8.14.3
- JUnit 5 with Spring Boot Test

## Java Version Requirements

**Critical:** Spring Boot 3.5.9 requires Java 17 or higher. This project uses Java 21.

On Windows systems with multiple Java versions, IntelliJ IDEA and command line may use different Java versions. If you encounter build errors about "JVM runtime version 17":

1. **For Gradle builds:** The ordering service is configured to use Java 21 via Java toolchain in `build.gradle`
2. **For IntelliJ IDEA:** Configure both Project SDK and Gradle JVM to Java 21:
   - `File → Project Structure → Project` - set SDK to Java 21
   - `File → Settings → Build Tools → Gradle` - set Gradle JVM to Java 21
   - `File → Invalidate Caches` to clear old configuration

## Build and Test Commands

### Ordering Service

Navigate to the service directory first:
```bash
cd microservices/ordering/algashop-ordering
```

**Build commands:**
```bash
# Build the project
./gradlew build

# Build without running tests
./gradlew build -x test

# Clean and rebuild
./gradlew clean build

# Run without daemon (useful for CI/CD)
./gradlew build --no-daemon
```

**Test commands:**
```bash
# Run all tests
./gradlew test

# Run tests with detailed output
./gradlew test --info

# Run specific test class
./gradlew test --tests CustomerTest

# Run specific test method
./gradlew test --tests CustomerTest.shouldThrowExceptionWhenEmailIsInvalid

# Generate test report (available at build/reports/tests/test/index.html)
./gradlew test
```

**Running the application:**
```bash
# Run Spring Boot application
./gradlew bootRun

# Build and run JAR
./gradlew bootJar
java -jar build/libs/ordering-0.0.1-SNAPSHOT.jar
```

**Useful Gradle tasks:**
```bash
# List all available tasks
./gradlew tasks

# Check dependencies
./gradlew dependencies

# View project info
./gradlew properties
```

## Domain-Driven Design Architecture

The ordering service implements DDD principles with clear bounded contexts:

**Package structure:**
- `domain/entity/` - Aggregate roots (e.g., Customer)
- `domain/exception/` - Domain-specific exceptions
- `domain/validator/` - Domain validation logic
- `domain/utility/` - Domain utilities (ID generation)

**Key patterns:**
- **Aggregate Roots** enforce business invariants (`Customer` is complete; `Order` is scaffolded — draft factory + getters only, lifecycle behaviors not yet implemented)
- **Domain Exceptions** represent business rule violations
- **Value Objects** encapsulate domain concepts (e.g. `Email`, `Money`, `Address`, `FullName`, typed IDs under `valueobject/id/`)
- Validation logic in `domain/validator/FieldValidations` builds on `commons-validator`

**ID generation (`domain/utility/IdGenerator`):**
- `generateTimeBasedUUID()` — time-based UUIDv7-style UUID via `java-uuid-generator`, used for entity IDs
- `gererateTSID()` — TSID via `hypersistence-tsid`; node/count configured through `TSID_NODE` / `TSID_NODE_COUNT` env vars for distributed generation

**Order lifecycle (`OrderStatus`):** `DRAFT -> PLACED -> PAID -> READY`, with `CANCELED` as a terminal state from any point (enum only today — transition rules not yet enforced in `Order`).

**Customer Entity Business Rules:**
- Archived customers cannot be modified (throws `CustomerArchivedException`)
- Email must be valid RFC-compliant format
- Birth date must be in the past
- Loyalty points can only increase
- Personal data is anonymized on archival (GDPR compliance)

## Working with Submodules

When making changes to microservices:

1. **Navigate to the submodule directory:**
   ```bash
   cd microservices/ordering/algashop-ordering
   ```

2. **Work as in a normal Git repository:**
   ```bash
   git checkout -b feature/my-feature
   # Make changes
   git add .
   git commit -m "feat: add new feature"
   git push origin feature/my-feature
   ```

3. **Update meta-repository to reference new commit:**
   ```bash
   cd ../../..  # Back to meta-repo root
   git add microservices/ordering
   git commit -m "chore: update ordering service to include new feature"
   git push
   ```

## Key Configuration Files

- **build.gradle** - Dependencies, plugins, and Java version configuration
- **application.properties** - Spring Boot application configuration (in `src/main/resources/`)
- **settings.gradle** - Project name and module configuration
- **.gitmodules** - Submodule repository references

## Adding New Microservices

To add a new microservice as a submodule:

```bash
# From meta-repository root
git submodule add <repository-url> microservices/<service-name>
git commit -m "chore: add <service-name> microservice"
```

Follow the same patterns as the ordering service:
- Use Spring Boot 3.5.9+ with Java 21
- Implement DDD principles for domain logic
- Use Gradle with wrapper for builds
- Write tests using JUnit 5
