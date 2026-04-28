# Environment Standards

> Applicable mode: Development mode
> Rule prefix: R-ENV

## R-ENV-01 Environment variable file path

Environment variable configuration file fixed path:
```
project-root/project.env.yaml
```

## R-ENV-02 Use YAML format

Environment variable file uses YAML format with the following structure:

```yaml
JAVA_HOME:
  PATH: "C:\\Program Files\\Java\\jdk-17"
  DESCRIPTION: "JDK installation path"

MAVEN_HOME:
  PATH: "C:\\apache-maven-3.9.0"
  DESCRIPTION: "Maven installation path"

MYSQL:
  HOST: "localhost"
  PORT: 3306
  DATABASE: "test_db"
  USERNAME: "root"
  PASSWORD: "${MYSQL_PASSWORD}"  # sensitive values use placeholders, actual values injected via environment variables
  DESCRIPTION: "MySQL database connection info"

REDIS:
  HOST: "localhost"
  PORT: 6379
  DESCRIPTION: "Redis connection info"
```

Each environment variable contains:
- `PATH` / `HOST` / `PORT` and other specific config items
- `DESCRIPTION`: purpose description

## R-ENV-03 Auto-create and prompt

When `project.env.yaml` does not exist:

1. **Auto-create file**
2. **Determine required environments based on project characteristics**:
   - Detected `pom.xml` → prompt to configure `JAVA_HOME`, `MAVEN_HOME`
   - Detected `spring-boot` dependency → additionally prompt `MYSQL`, `REDIS`, etc.
   - Detected `Dockerfile` → additionally prompt `DOCKER_REGISTRY`
3. **Output prompt message**:

```
project.env.yaml not detected, template file auto-created.
Based on project analysis, please complete the following environment variables:

[Required]
- JAVA_HOME.PATH: JDK installation path (pom.xml detected)
- MAVEN_HOME.PATH: Maven installation path (pom.xml detected)

[Optional]
- MYSQL.HOST: Database address (spring-boot-starter-jdbc dependency detected)
- REDIS.HOST: Redis address (spring-boot-starter-data-redis dependency detected)

Please fill in project.env.yaml and let me know when done.
```

Wait for the user to fill in before continuing.

## Appendix: Complete project.env.yaml Example

```yaml
# ============================================
# Project Environment Variables Configuration
# Notes:
#   1. Sensitive info uses ${ENV_VAR} placeholder
#   2. When adding new env vars, please also add DESCRIPTION
# ============================================

JAVA_HOME:
  PATH: "${JAVA_HOME}"
  DESCRIPTION: "JDK 17 installation path"

MAVEN_HOME:
  PATH: "${MAVEN_HOME}"
  DESCRIPTION: "Maven 3.9 installation path"

MYSQL:
  HOST: "${MYSQL_HOST:localhost}"
  PORT: ${MYSQL_PORT:3306}
  DATABASE: "${MYSQL_DATABASE:test}"
  USERNAME: "${MYSQL_USERNAME:root}"
  PASSWORD: "${MYSQL_PASSWORD:}"
  DESCRIPTION: "Development environment MySQL"

REDIS:
  HOST: "${REDIS_HOST:localhost}"
  PORT: ${REDIS_PORT:6379}
  PASSWORD: "${REDIS_PASSWORD:}"
  DESCRIPTION: "Development environment Redis"

THREAD_POOL:
  CORE_SIZE: ${THREAD_CORE_SIZE:}
  MAX_SIZE: ${THREAD_MAX_SIZE:}
  QUEUE_CAPACITY: ${THREAD_QUEUE_CAPACITY:}
  DESCRIPTION: "Async thread pool params (leave empty for auto-tuning)"
```
