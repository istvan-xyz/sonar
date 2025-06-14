# SonarQube Setup

This directory contains a Docker Compose setup for running SonarQube locally for code quality analysis.

## Quick Start

1. **Start SonarQube:**

    ```bash
    docker-compose up -d
    ```

2. **Access SonarQube:**

    - Open your browser to http://localhost:9000
    - Default credentials: `admin` / `admin`
    - You'll be prompted to change the password on first login

3. **Stop SonarQube:**
    ```bash
    docker-compose down
    ```

## Services

-   **SonarQube Community Edition** - Static code analysis server
-   **PostgreSQL 15** - Database backend for persistent storage

## Configuration

### Ports

-   `9000` - SonarQube web interface

### Volumes

-   `sonarqube_data` - SonarQube application data
-   `sonarqube_extensions` - Plugins and extensions
-   `sonarqube_logs` - Application logs
-   `sonarqube_db` - PostgreSQL database data

### Environment Variables

-   `SONAR_JDBC_URL` - Database connection string
-   `SONAR_JDBC_USERNAME` - Database username
-   `SONAR_JDBC_PASSWORD` - Database password

## Usage

### View Logs

```bash
# All services
docker-compose logs -f

# SonarQube only
docker-compose logs -f sonarqube

# Database only
docker-compose logs -f sonarqube-db
```

### Restart Services

```bash
docker-compose restart
```

### Update Images

```bash
docker-compose pull
docker-compose up -d
```

## Project Analysis

### Create a New Project

1. Log into SonarQube at http://localhost:9000
2. Click "Create Project" → "Manually"
3. Set project key and display name
4. Generate a project token
5. Choose your analysis method (CLI, CI/CD, etc.)

### Analyze with SonarQube Scanner

```bash
# Install SonarQube Scanner CLI
npm install -g sonarqube-scanner

# Run analysis (from your project root)
sonar-scanner \
  -Dsonar.projectKey=your-project-key \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=your-project-token
```

### Language-Specific Analysis

#### TypeScript/JavaScript

```bash
# Install SonarJS plugin (if not already installed)
# Add to your project's sonar-project.properties:
sonar.projectKey=my-typescript-project
sonar.sources=src
sonar.exclusions=**/*.test.ts,**/*.spec.ts,node_modules/**
sonar.typescript.lcov.reportPaths=coverage/lcov.info
```

#### Python

```bash
# Add to sonar-project.properties:
sonar.projectKey=my-python-project
sonar.sources=.
sonar.python.coverage.reportPaths=coverage.xml
sonar.exclusions=**/*_test.py,**/test_*.py,venv/**
```

## Troubleshooting

### Elasticsearch Memory Issues

If you encounter the error: `max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

**Solution 1 - Docker Compose (Recommended):**
The `docker-compose.yml` file already includes the required system configuration:
```yaml
sysctls:
  - vm.max_map_count=262144
```

**Solution 2 - Host System Configuration:**
If the Docker approach doesn't work, set it on the host:

```bash
# Temporary fix (until reboot)
sudo sysctl -w vm.max_map_count=262144

# Permanent fix
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**For macOS users:**
The sysctls configuration in Docker Compose should work automatically. If issues persist, try:
```bash
# Check current value
docker run --rm alpine sysctl vm.max_map_count

# Restart Docker Desktop if needed
```

### SonarQube Won't Start

-   Check available memory (requires at least 2GB RAM)
-   Verify Docker has sufficient resources allocated
-   Check logs: `docker-compose logs sonarqube`

### Database Connection Issues

-   Ensure PostgreSQL container is healthy: `docker-compose ps`
-   Check database logs: `docker-compose logs sonarqube-db`

### Permission Issues

```bash
# Fix volume permissions if needed
sudo chown -R 999:999 volumes/sonarqube_data
```

### Reset Everything

```bash
# Stop and remove all containers and volumes
docker-compose down -v
docker-compose up -d
```

## Useful Commands

```bash
# Check container status
docker-compose ps

# Execute commands in SonarQube container
docker-compose exec sonarqube bash

# Access PostgreSQL
docker-compose exec sonarqube-db psql -U sonar -d sonar

# Monitor resource usage
docker stats
```

## Security Notes

-   Change default admin password immediately after first login
-   Consider using environment variables for sensitive configuration
-   For production use, configure HTTPS and proper authentication
-   Regularly update SonarQube and PostgreSQL images

## Links

-   [SonarQube Documentation](https://docs.sonarsource.com/sonarqube/)
-   [SonarQube Scanner CLI](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/)
-   [Docker Hub - SonarQube](https://hub.docker.com/_/sonarqube)
