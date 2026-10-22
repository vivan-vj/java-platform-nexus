# java-platform-nexus

A centralized Maven platform for managing dependency versions, build plugin configuration,
and Java compatibility baselines across enterprise Spring Boot applications.

Designed for organizations with many Java repositories. Provides two independent
consumption models so any application can adopt the platform regardless of its
existing parent POM constraints.

---

## Architecture Overview

```
java-platform-nexus/                  ← Root reactor (aggregator only)
├── java-platform-nexus-bom/          ← Single source of truth for dependency versions
└── java-platform-nexus-parent/       ← Build defaults + BOM import
```

The root aggregator is never used as a parent by consuming applications.
See [docs/architecture.md](docs/architecture.md) for full module descriptions.

---

## Maven Coordinates

| Module | groupId | artifactId | Version |
|--------|---------|------------|---------|
| Root aggregator | `com.example.platform` | `java-platform-nexus` | `1.0.0-SNAPSHOT` |
| Platform BOM | `com.example.platform` | `java-platform-nexus-bom` | `1.0.0-SNAPSHOT` |
| Platform Parent | `com.example.platform` | `java-platform-nexus-parent` | `1.0.0-SNAPSHOT` |

> **Group ID**: `com.example.platform` is a placeholder. Replace it with your
> organization's namespace before publishing. Update all three POMs consistently.

---

## Consumption Models

### Model 1 — Parent Inheritance

Use when the application has no mandatory corporate parent.
Inherits dependency version management **and** build plugin configuration.

```xml
<project>
  <parent>
    <groupId>com.example.platform</groupId>
    <artifactId>java-platform-nexus-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>

  <groupId>com.acme.myapp</groupId>
  <artifactId>my-service</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencies>
    <!-- No version needed — managed by the platform BOM via the parent -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
    </dependency>
  </dependencies>
</project>
```

To target Java 21 instead of the default Java 17:

```xml
<properties>
  <java.version>21</java.version>
</properties>
```

### Model 2 — BOM Import Only

Use when the application already has a mandatory parent it cannot change.
Gains dependency version management only. Plugin configuration, compiler settings,
and Enforcer rules are **not** inherited — the application's own parent must supply those.

```xml
<project>
  <!-- Keep your existing parent -->
  <parent>
    <groupId>com.acme.corp</groupId>
    <artifactId>acme-corporate-parent</artifactId>
    <version>2.1.0</version>
  </parent>

  <groupId>com.acme.myapp</groupId>
  <artifactId>my-service</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.example.platform</groupId>
        <artifactId>java-platform-nexus-bom</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <!-- No version needed — managed by the platform BOM -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
</project>
```

---

## Java and Framework Compatibility

The current platform release line targets **Spring Boot 3.3.x** with a **Java 17 minimum**.

| Java | Spring Boot | Status |
|------|-------------|--------|
| 17 | 3.3.4 | ✅ Verified baseline |
| 21 | 3.3.4 | ⚠️ Unverified — expected compatible |
| 11 | 3.3.4 | ❌ Incompatible — Spring Boot 3.x requires Java 17+ |

See [docs/compatibility-matrix.md](docs/compatibility-matrix.md) for the full matrix
including Spring Cloud, database drivers, and cloud SDK combinations.

> The JDK used to run Maven must be >= the `java.version` property configured in your
> application POM (default: 17).

---

## Dependency Version Policy

- All dependency versions are managed in `java-platform-nexus-bom`.
- No version is duplicated between the BOM and the parent.
- Upstream BOMs are imported in a documented precedence order; `spring-boot-dependencies` wins.
- Explicit version overrides require a documented reason in the BOM XML.
- Version ranges and dynamic qualifiers are prohibited.

See [docs/dependency-policy.md](docs/dependency-policy.md) for the full policy.

---

## Build and Validation

### Prerequisites

- JDK 17 or higher
- Maven 3.9.0 or higher

### Build the platform locally

```bash
# From the repository root
mvn install
```

### Validate POM structure

```bash
mvn validate
```

### Inspect resolved versions

```bash
# Effective POM for the parent (shows all resolved plugin and dependency versions)
mvn help:effective-pom -pl java-platform-nexus-parent

# Effective POM for the BOM
mvn help:effective-pom -pl java-platform-nexus-bom
```

### Verify a consuming application's dependency tree

```bash
# Run from the consuming application's directory
mvn dependency:tree
```

---

## Release and Publishing

### Versioning

This platform follows semantic versioning:

- **Patch** (`1.0.x`): dependency version bumps, CVE fixes, documentation updates
- **Minor** (`1.x.0`): new managed dependencies, new plugin defaults, non-breaking changes
- **Major** (`x.0.0`): breaking changes, new Spring Boot baseline, Java minimum bump

### Publishing

Before publishing, configure `distributionManagement` in the root `pom.xml`:

- **Maven Central**: requires Sonatype OSSRH account, GPG signing (`maven-gpg-plugin`),
  and `nexus-staging-maven-plugin`.
- **Private Nexus/Artifactory**: set `platform.releases.url` and `platform.snapshots.url`
  in your `~/.m2/settings.xml` or CI environment variables.

```bash
# Deploy a snapshot
mvn deploy

# Release (after configuring maven-release-plugin or your CI release process)
mvn release:prepare release:perform
```

### Immutability

Once a release version is published, it must not be modified.
Use a new patch version for any change, including documentation.

---

## Contribution Guidelines

1. All dependency additions require a rationale comment in the BOM and an entry in
   [docs/dependency-policy.md](docs/dependency-policy.md).
2. All version changes must be reflected in [docs/compatibility-matrix.md](docs/compatibility-matrix.md)
   with an honest status (do not mark unverified combinations as verified).
3. No version may be declared in both the BOM and the parent.
4. Plugin versions belong in the parent only, never in the BOM.
5. Run `mvn validate` and `mvn verify` before opening a pull request.
6. Breaking changes require a major version bump and release notes.
