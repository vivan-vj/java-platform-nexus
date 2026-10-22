# Platform Architecture

## Module Overview

```
java-platform-nexus/                  ← Root reactor (aggregator only)
├── java-platform-nexus-bom/          ← Dependency version management
└── java-platform-nexus-parent/       ← Build configuration + BOM import
```

## Root Aggregator (`java-platform-nexus`)

Exists solely to build all platform modules together with a single `mvn install`.
It is **never** used as a parent by consuming applications.
It carries no dependency management and no plugin configuration of its own.
Its only responsibilities are declaring the `<modules>` list and holding shared
properties (`project.build.sourceEncoding`, `maven.minimum.version`).

## Platform BOM (`java-platform-nexus-bom`)

The single source of truth for all managed dependency versions.

- Imports upstream BOMs (`spring-boot-dependencies`, `spring-cloud-dependencies`, etc.)
- Explicitly manages versions not covered by upstream BOMs
- Can be imported independently by any Maven project via `<scope>import</scope>`
- Importing the BOM does **not** add any dependency to the consuming project's classpath
- Importing the BOM does **not** inherit plugin configuration, compiler settings,
  build properties, or Enforcer rules

## Platform Parent (`java-platform-nexus-parent`)

The recommended parent for new applications.

- Imports `java-platform-nexus-bom` via `dependencyManagement` — no version duplication
- Provides `pluginManagement` for all standard Maven lifecycle plugins
- Activates Maven Enforcer on the `validate` phase
- Exposes `java.version` property for applications to override the compiler target
- Does **not** duplicate any version already managed in the BOM

## Consumption Models

### Model 1 — Parent Inheritance

Use when the application has no mandatory corporate parent.

```xml
<parent>
  <groupId>com.example.platform</groupId>
  <artifactId>java-platform-nexus-parent</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</parent>

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
```

The application inherits dependency version management **and** build plugin configuration.

### Model 2 — BOM Import Only

Use when the application already has a mandatory parent it cannot change.

```xml
<!-- Keep your existing parent -->
<parent>
  <groupId>com.acme.corp</groupId>
  <artifactId>acme-corporate-parent</artifactId>
  <version>2.1.0</version>
</parent>

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
```

The application retains its own parent and gains only dependency version management.
Plugin configuration, compiler settings, and Enforcer rules are **not** inherited.
The application's own parent or its own `<build>` section must supply those.

## Design Principles

- No version is declared in both the BOM and the parent
- No dependency is added to consuming applications automatically
- Plugin versions are pinned; no plugin uses a floating or range version
- The platform is extended by adding new modules to the reactor, not by
  growing the BOM or parent unboundedly
- All version properties live in the BOM; the parent references only plugin versions
