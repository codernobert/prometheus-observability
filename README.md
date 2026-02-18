# Prometheus Observability

A Spring Boot application configured for Prometheus metrics monitoring and observability. This application exposes metrics in Prometheus format through Spring Boot Actuator endpoints.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Building the Project](#building-the-project)
- [Running the Application](#running-the-application)
- [API Endpoints](#api-endpoints)
- [Configuration](#configuration)
- [Prometheus Integration](#prometheus-integration)
- [Troubleshooting](#troubleshooting)

## Overview

This project is a Spring Boot 4.0.2 application that:
- Provides REST API endpoints
- Exposes application metrics via Spring Boot Actuator
- Integrates with Prometheus for metrics scraping
- Uses Micrometer for metrics collection
- Runs on port 8080

The application is designed to be monitored by Prometheus for observability and includes all necessary components for metrics collection and exposure.

## Prerequisites

- Java Development Kit (JDK) 17 or higher
- Maven 3.6.0 or higher
- Prometheus (optional, for metrics scraping)
- Spring Boot 4.0.2

## Project Structure

```
prometheus-observability/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/config/prometheusobservability/
│       │       └── PrometheusObservabilityApplication.java
│       └── resources/
│           └── application.properties
├── pom.xml
├── prometheus.yml
├── README.md
└── target/
    └── prometheus-observability-0.0.1-SNAPSHOT.jar
```

### Key Components

- **PrometheusObservabilityApplication.java** - Main Spring Boot application class
- **pom.xml** - Maven project configuration with dependencies
- **application.properties** - Application configuration including server port and management endpoints
- **prometheus.yml** - Prometheus configuration file for scraping metrics

## Building the Project

### Using Maven

```bash
# Clean and build the project
mvn clean install

# Build without running tests
mvn clean install -DskipTests

# Compile only
mvn compile
```

### Build Output

The built JAR file will be located at:
```
target/prometheus-observability-0.0.1-SNAPSHOT.jar
```

## Running the Application

### Option 1: Using Maven Spring Boot Plugin

```bash
mvn spring-boot:run
```

### Option 2: Running the JAR File Directly

```bash
java -jar target/prometheus-observability-0.0.1-SNAPSHOT.jar
```

### Option 3: Using PowerShell Script (Windows)

```powershell
PowerShell -ExecutionPolicy Bypass -File .\start-app.ps1
```

### Startup Output

When the application starts successfully, you should see:

```
Tomcat initialized with port 8080 (http)
...
Exposing 2 endpoints beneath base path '/actuator'
```

The application will be available at: `http://localhost:8080`

## API Endpoints

### Base URL
```
http://localhost:8080
```

### Management Endpoints

The application exposes the following management endpoints via Spring Boot Actuator:

#### Prometheus Metrics Endpoint
```
GET http://localhost:8080/actuator/prometheus
```

Returns all application metrics in Prometheus format (text/plain; version=0.0.4).

**Example Response:**
```
# HELP tomcat_sessions_created_sessions_total  
# TYPE tomcat_sessions_created_sessions_total counter
tomcat_sessions_created_sessions_total 0.0
# HELP tomcat_sessions_alive_max_sessions  
# TYPE tomcat_sessions_alive_max_sessions gauge
tomcat_sessions_alive_max_sessions 0.0
...
```

#### Actuator Endpoints List
```
GET http://localhost:8080/actuator
```

Returns a list of available actuator endpoints.

## Configuration

### Application Properties

Located in: `src/main/resources/application.properties`

```properties
# Application name
spring.application.name=prometheus-observability

# Server port
server.port=8080

# Management endpoints exposure
management.endpoints.web.exposure.include=prometheus
management.endpoint.prometheus.enabled=true
```

### Customizing Configuration

To change the server port, update the `application.properties` file:

```properties
server.port=9090
```

To expose additional actuator endpoints:

```properties
management.endpoints.web.exposure.include=prometheus,metrics,health,info
```

## Prometheus Integration

### Prometheus Configuration

The `prometheus.yml` file configures Prometheus to scrape metrics from this application:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'render-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['https://config-server-service-2.onrender.com/']
```

### Setting Up Prometheus

1. **Download Prometheus** from https://prometheus.io/download/

2. **Update prometheus.yml** with your application's URL:
   ```yaml
   scrape_configs:
     - job_name: 'prometheus-observability'
       metrics_path: '/actuator/prometheus'
       static_configs:
         - targets: ['http://localhost:8080']
   ```

3. **Start Prometheus**:
   ```bash
   ./prometheus --config.file=prometheus.yml
   ```

4. **Access Prometheus UI** at: `http://localhost:9090`

### Querying Metrics

Once Prometheus is scraping metrics, you can query them using PromQL:

```promql
# JVM Memory Usage
jvm_memory_used_bytes

# HTTP Request Count
http_server_requests_seconds_count

# Tomcat Sessions
tomcat_sessions_alive_max_sessions
```

## Project Dependencies

### Core Dependencies

- **spring-boot-starter-web** (4.0.2) - Web and REST API support
- **spring-boot-starter-actuator** (4.0.2) - Application monitoring and management endpoints
- **micrometer-registry-prometheus** - Prometheus metrics registry for Micrometer

### Plugins

- **spring-boot-maven-plugin** (4.0.2) - Maven plugin for Spring Boot packaging and execution

## Available Metrics

The application exposes the following default metrics:

### JVM Metrics
- JVM memory usage
- Thread count and status
- Garbage collection statistics
- Class loading statistics

### Tomcat Metrics
- Active connections
- Request count and latency
- Session statistics
- Thread pool metrics

### Application Metrics
- Custom business metrics (if implemented)
- HTTP request metrics
- Database connection pool metrics (if applicable)

## Troubleshooting

### Port Already in Use

**Error:** `Port 8080 was already in use`

**Solution:**
- Kill the existing Java process:
  ```powershell
  Get-Process | Where-Object {$_.ProcessName -match "java"} | Stop-Process -Force
  ```
- Or change the port in `application.properties`:
  ```properties
  server.port=8081
  ```

### Metrics Endpoint Returns 404

**Error:** `No explicit mapping for /error` (404 error)

**Solution:**
- Ensure `spring-boot-starter-actuator` dependency is included in pom.xml
- Verify `management.endpoints.web.exposure.include=prometheus` is set in application.properties
- Rebuild the project: `mvn clean install`

### Application Fails to Start

**Error:** `Failed to start bean 'webServerStartStop'`

**Solution:**
- Check that port 8080 is available
- Review application logs for detailed error messages
- Ensure Java 17+ is installed: `java -version`

### Maven Build Issues

**Solution:**
- Clean Maven cache: `mvn clean`
- Update dependencies: `mvn dependency:resolve`
- Verify Java version: `javac -version`

## Development

### Adding Custom Metrics

To add custom business metrics, create a @Component that uses MeterRegistry:

```java
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Component;

@Component
public class CustomMetrics {
    private final MeterRegistry meterRegistry;

    public CustomMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void recordCustomMetric(String metricName, double value) {
        meterRegistry.gauge(metricName, value);
    }
}
```

### Extending Configuration

Modify `application.properties` or use environment variables:

```bash
MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE=prometheus,metrics,health,info
```

## Deployment

### Build for Production

```bash
mvn clean install -DskipTests
```

### Docker Deployment (Optional)

Create a `Dockerfile`:

```dockerfile
FROM openjdk:17-jdk-slim
COPY target/prometheus-observability-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build and run:
```bash
docker build -t prometheus-observability .
docker run -p 8080:8080 prometheus-observability
```

## License

This project is provided as-is.

## Support

For issues, refer to:
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Micrometer Documentation](https://micrometer.io/)
- [Prometheus Documentation](https://prometheus.io/docs)

---

**Version:** 0.0.1-SNAPSHOT  
**Spring Boot:** 4.0.2  
**Java:** 17  
**Last Updated:** February 2026

