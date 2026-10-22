# Compatibility Matrix

## Status Definitions

| Status | Meaning |
|--------|---------|
| ✅ Verified | Tested combination confirmed to work |
| ⚠️ Unverified | Expected to work based on upstream documentation; not yet tested on this platform |
| ❌ Incompatible | Known to be incompatible |
| 🚧 Planned | Planned for a future platform release line; not yet implemented |

---

## Platform Release Line 1.x — Spring Boot 3.3.x Baseline

### Build Toolchain

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| Java (minimum) | 17 | ✅ Verified | Spring Boot 3.x hard minimum; enforced by platform Enforcer rule |
| Java 21 (LTS) | 21 | ⚠️ Unverified | Spring Boot 3.3.x supports Java 21 per upstream docs; validate with your app |
| Java 24 | 24 | ⚠️ Unverified | Not yet validated on this platform |
| Java 25 | 25 | 🚧 Planned | Pending GA release and Spring Boot support confirmation |
| Maven | 3.9.x | ✅ Verified | Minimum enforced by platform Enforcer rule |

### Spring Ecosystem

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| Spring Boot | 3.3.4 | ✅ Verified | Platform baseline |
| Spring Framework | 6.1.x | ✅ Verified | Managed transitively by `spring-boot-dependencies` |
| Spring Cloud (Leyton) | 2023.0.3 | ⚠️ Unverified | Compatible with Spring Boot 3.3.x per Spring Cloud release train docs |
| Spring AI | 1.0.0-M3 | ⚠️ Unverified | Milestone release; API not yet stable; requires Spring Boot 3.3+ |

### Database and Persistence

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| Spring Data JPA | managed by spring-boot-dependencies | ✅ Verified | |
| MySQL Connector/J | 8.4.0 | ⚠️ Unverified | Requires MySQL 8.x server |
| PostgreSQL JDBC | 42.7.4 | ⚠️ Unverified | |
| Flyway Core | 10.18.0 | ⚠️ Unverified | Flyway 10.x requires Java 17+; incompatible with Java 11 |
| Flyway MySQL module | 10.18.0 | ⚠️ Unverified | Required in addition to flyway-core for MySQL |
| Flyway PostgreSQL module | 10.18.0 | ⚠️ Unverified | Required in addition to flyway-core for PostgreSQL |
| Liquibase | 4.29.2 | ⚠️ Unverified | |

### Cloud — Azure

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| Azure SDK BOM | 1.2.27 | ⚠️ Unverified | |
| Spring Cloud Azure | 5.17.0 | ⚠️ Unverified | Requires Spring Boot 3.x |

### Cloud — AWS

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| AWS SDK v2 BOM | 2.28.3 | ⚠️ Unverified | |
| Spring Cloud AWS | 3.2.1 | ⚠️ Unverified | Requires Spring Boot 3.x |

### Observability

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| Micrometer | managed by spring-boot-dependencies | ✅ Verified | No separate version needed |
| OpenTelemetry SDK | 1.42.1 | ⚠️ Unverified | |
| OpenTelemetry Instrumentation | 2.9.0 | ⚠️ Unverified | |

### Testing

| Component | Version | Status | Notes |
|-----------|---------|--------|-------|
| JUnit Jupiter | managed by spring-boot-dependencies | ✅ Verified | |
| Mockito | managed by spring-boot-dependencies | ✅ Verified | |
| Testcontainers | 1.20.2 | ⚠️ Unverified | Also managed by spring-boot-dependencies 3.1+ |

---

## Known Incompatibilities

| Combination | Status | Notes |
|-------------|--------|-------|
| Flyway 10.x + Java 11 | ❌ Incompatible | Flyway 10.x requires Java 17 minimum |
| Spring Boot 3.x + Java 11 | ❌ Incompatible | Spring Boot 3.x requires Java 17 minimum |
| Spring Cloud Azure 5.x + Spring Boot 2.x | ❌ Incompatible | Spring Cloud Azure 5.x requires Spring Boot 3.x |
| Spring Cloud AWS 3.x + Spring Boot 2.x | ❌ Incompatible | Spring Cloud AWS 3.x requires Spring Boot 3.x |

---

## Future Release Lines

### Platform Release Line 2.x — Spring Boot 2.7.x / Java 11 Baseline

> 🚧 Not yet implemented.

Spring Boot 2.7.x reached end-of-life in November 2023. A separate release line
targeting Java 11 + Spring Boot 2.7.x would require:

- A separate set of BOM version properties
- Flyway 9.x (last version supporting Java 11)
- Spring Cloud 2021.0.x (Jubilee)
- Spring Cloud Azure 4.x
- Spring Cloud AWS 2.x

This release line will only be created if there is a validated business need.
It will not share a BOM or parent with the 1.x line.

---

## Validation Commands

Run these commands to verify the platform builds correctly:

```bash
# Validate all POM files are well-formed and reactor resolves
mvn validate

# Full build including dependency resolution
mvn verify

# Inspect effective POM for the parent (shows resolved plugin and dependency versions)
mvn help:effective-pom -pl java-platform-nexus-parent

# Inspect effective POM for the BOM
mvn help:effective-pom -pl java-platform-nexus-bom
```
