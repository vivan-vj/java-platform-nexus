# Dependency Policy

## Upstream BOM Import Precedence

When two imported BOMs manage the same artifact, Maven uses the **first declaration**
in `dependencyManagement`. The platform BOM imports BOMs in this order:

| Priority | BOM | Artifact |
|----------|-----|---------|
| 1 (highest) | `spring-boot-dependencies` | `org.springframework.boot:spring-boot-dependencies` |
| 2 | `spring-cloud-dependencies` | `org.springframework.cloud:spring-cloud-dependencies` |
| 3 | `spring-ai-bom` | `org.springframework.ai:spring-ai-bom` |
| 4 | `azure-sdk-bom` | `com.azure:azure-sdk-bom` |
| 5 | `spring-cloud-azure-dependencies` | `com.azure.spring:spring-cloud-azure-dependencies` |
| 6 | AWS SDK v2 BOM | `software.amazon.awssdk:bom` |
| 7 | Spring Cloud AWS BOM | `io.awspring.cloud:spring-cloud-aws-dependencies` |
| 8 | `opentelemetry-bom` | `io.opentelemetry:opentelemetry-bom` |
| 9 | `opentelemetry-instrumentation-bom` | `io.opentelemetry.instrumentation:opentelemetry-instrumentation-bom` |

Spring Boot's BOM has the highest precedence because it defines the tested dependency
combination for the Spring ecosystem. Overriding its versions without a documented
compatibility reason is prohibited.

## Explicit Version Overrides

A dependency version is declared explicitly in the BOM (outside a BOM import) only when:

1. It is **not** managed by any imported upstream BOM, **or**
2. A specific version is required for a documented security or compatibility reason.

Every explicit override must include an XML comment stating the reason.

### Current Overrides

| Artifact | Reason |
|----------|--------|
| `com.mysql:mysql-connector-j` | Align with MySQL 8.x server; spring-boot-dependencies version acceptable as fallback |
| `org.postgresql:postgresql` | Align with PostgreSQL 16.x server |
| `org.flywaydb:flyway-*` | Ensure all Flyway modules (core, mysql, postgresql) stay on the same version |
| `org.liquibase:liquibase-core` | Align with Spring Boot 3.3.x tested combination |
| `org.testcontainers:testcontainers-bom` | Allow CVE-driven override before next Spring Boot release |

## Application-Level Overrides

Consuming applications may override a managed version by declaring it in their own
`dependencyManagement` section. This is permitted but must be:

- Documented in the application's own changelog or ADR
- Reviewed when the platform version is upgraded
- Removed once the platform catches up to the required version

Applications must **not** use version ranges (`[1.0,2.0)`) or `LATEST`/`RELEASE` qualifiers.

## Dependency Convergence

The Maven Enforcer `dependencyConvergence` rule is **not** enabled by default.
Imported upstream BOMs (particularly Spring Boot and Spring Cloud) intentionally
manage overlapping transitive dependencies, and the rule produces false positives
in that context.

Teams that require strict convergence may add the rule to their own application POM:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <configuration>
    <rules>
      <dependencyConvergence/>
    </rules>
  </configuration>
</plugin>
```

## Security Vulnerability Remediation

When a CVE is identified in a managed dependency:

1. Assess whether the upstream BOM has already addressed it in a newer version.
2. If yes: raise a platform version bump PR targeting the next patch release.
3. If no: add an explicit override in the BOM with a comment referencing the CVE number.
4. Publish a patch release of the platform (e.g. `1.0.1`).
5. Remove the override once the upstream BOM resolves it and the platform upgrades.

## Adding New Dependencies

A dependency is added to the platform BOM only when:

- It is used by **multiple** consuming applications, **and**
- Its version has been validated against the current compatibility baseline, **and**
- A rationale is documented in this file or in the PR description.

Do not add a dependency merely because it is popular or might be useful.

## Removing or Deprecating Managed Dependencies

1. Mark the dependency with a comment `<!-- DEPRECATED: removal planned in vX.Y -->`.
2. Communicate the removal in release notes with at least one minor version of notice.
3. Remove in the next major version.

## Version Ranges and Dynamic Versions

Version ranges and dynamic qualifiers (`LATEST`, `RELEASE`, `SNAPSHOT` in release builds)
are prohibited in the platform BOM and in consuming application POMs for production releases.
SNAPSHOT versions are permitted only during active development of the platform itself.

## Plugin Version Policy

Plugin versions are managed exclusively in `java-platform-nexus-parent` via `pluginManagement`.
They are never declared in the BOM. This separation ensures that BOM-only consumers
are not affected by plugin version decisions.
