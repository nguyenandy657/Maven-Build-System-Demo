# Word Frequency Analyzer — Maven Build System Demo

<!-- ── CI BADGES ─────────────────────────────────────────────────────── -->
<!-- These badges are auto-updated by the CI pipeline on every push.     -->
<!-- Replace YOUR_USERNAME/YOUR_REPO with your actual GitHub path.       -->

![Build Status](https://github.com/nguyenandy657/Maven-Build-System-Demo/actions/workflows/ci.yml/badge.svg)
![Coverage](https://github.com/nguyenandy657/Maven-Build-System-Demo/raw/gh-pages/.badges/jacoco.svg)
![Branches](https://github.com/nguyenandy657/Maven-Build-System-Demo/raw/gh-pages/.badges/branches.svg)

A simple Java project that demonstrates core Maven (build system) concepts and CI/CD with GitHub Actions.

## Project Structure (Maven Standard Layout)


```
Java/
├── pom.xml                                    # Project Object Model (the build file)
├── .github/workflows/ci.yml                   # CI pipeline (GitHub Actions)
├── src/
│   ├── main/
│   │   ├── java/com/example/wordfreq/        # Application source code
│   │   │   ├── App.java                       # Entry point (main class)
│   │   │   ├── CsvExporter.java               # CSV export (Commons CSV)
│   │   │   └── WordFrequencyAnalyzer.java     # Core logic
│   │   └── resources/
│   │       └── sample.txt                     # Bundled resource file
│   └── test/
│       └── java/com/example/wordfreq/
│           ├── CsvExporterTest.java           # CSV exporter tests
│           └── WordFrequencyAnalyzerTest.java # Analyzer tests (JUnit 5)
└── README.md
```

Maven enforces **convention over configuration** — if you follow this directory layout, everything works without extra setup.

## Maven Concepts Demonstrated

### 1. Project Coordinates (GAV)
Every Maven artifact is identified by **GroupId:ArtifactId:Version**:
```xml
<groupId>com.example</groupId>
<artifactId>word-frequency-analyzer</artifactId>
<version>1.0-SNAPSHOT</version>
```

### 2. Properties
Centralized variables — change a version in one place:
```xml
<properties>
    <gson.version>2.11.0</gson.version>
    <junit.version>5.10.3</junit.version>
</properties>
```

### 3. Dependencies & Scopes
Maven downloads jars and their transitive dependencies automatically:
- **compile scope** (default): Gson — available everywhere, included in the jar
- **test scope**: JUnit — only available during testing, never shipped

### 4. Build Lifecycle & Phases
Maven's default lifecycle runs phases **in order**. Running a later phase automatically runs all earlier ones:

```
validate → compile → test → package → verify → install → deploy
```

### 5. Plugins
Plugins do the actual work at each phase:
- **maven-compiler-plugin** — compiles `.java` → `.class`
- **maven-surefire-plugin** — runs unit tests
- **maven-jar-plugin** — packages classes into a `.jar` with a manifest
- **maven-shade-plugin** — creates a fat jar (via profile)

### 6. Profiles
Conditional build configurations activated on demand:
```bash
mvn package -P fatjar    # Activates the "fatjar" profile
```

### 7. Resource Filtering
Files in `src/main/resources` are bundled into the jar. With `<filtering>true</filtering>`, Maven replaces `${...}` placeholders in resource files.

## Continuous Integration (CI)

This project includes a GitHub Actions CI pipeline (`.github/workflows/ci.yml`) that runs automatically on every push and pull request.

### What the CI Pipeline Does

```
Push/PR to main
      │
      ▼
┌─────────────────────────┐
│  1. Checkout code       │  ← Get the latest code
│  2. Set up JDK 17       │  ← Reproducible environment
│  3. mvn clean verify    │  ← Compile → Test → Coverage → Package
│  4. Publish test results│  ← Show pass/fail on GitHub
│  5. Generate badge      │  ← Coverage % as SVG image
│  6. Publish badge       │  ← Push to gh-pages branch
└─────────────────────────┘
```

### CI Concepts Demonstrated

| Concept | How It's Implemented |
|---|---|
| **Automated trigger** | `on: push` / `on: pull_request` — no manual steps needed |
| **Reproducible environment** | `setup-java@v4` installs the exact JDK version every time |
| **Dependency caching** | `cache: 'maven'` — skips re-downloading jars on repeat runs |
| **Test execution** | Maven Surefire runs all JUnit 5 tests automatically |
| **Test reporting** | `dorny/test-reporter` shows results directly on the PR/commit |
| **Code coverage** | JaCoCo plugin instruments code and generates coverage reports |
| **Coverage badge** | `jacoco-badge-generator` creates an SVG badge from JaCoCo output |
| **Artifact publishing** | Badge pushed to `gh-pages` branch, referenced in README |

### Setup Instructions

After pushing this project to GitHub:

1. **Replace badge URLs** in this README — change `YOUR_USERNAME/YOUR_REPO` to your actual GitHub path (e.g., `johndoe/word-frequency-analyzer`)
2. **Push to main** — the workflow runs automatically
3. **Check the Actions tab** — see the pipeline run, test results, and logs
4. **Badges appear** — after the first successful run, coverage badges are published to `gh-pages`

### JaCoCo Coverage Plugin (pom.xml)

JaCoCo hooks into the Maven lifecycle in two phases:
```xml
<!-- 1. Before tests: instruments bytecode to track which lines are executed -->
<goal>prepare-agent</goal>

<!-- 2. After tests (verify phase): generates HTML + CSV + XML reports -->
<goal>report</goal>
```

After running `mvn verify`, find the coverage report at:
```
target/site/jacoco/index.html   ← Open in browser for detailed coverage view
```

## Commands to Try

```bash
# Compile the source code (runs: validate → compile)
mvn compile

# Run unit tests (runs: validate → compile → test)
mvn test

# Package into a jar (runs: validate → compile → test → package)
mvn package

# Full build with all checks
mvn clean verify

# Build a fat jar (all dependencies bundled in one jar)
mvn clean package -P fatjar

# Run the application (after packaging)
java -cp target/word-frequency-analyzer-1.0-SNAPSHOT.jar:$(mvn dependency:build-classpath -q -DincludeScope=runtime -Dmdep.outputFile=/dev/stderr 2>&1) com.example.wordfreq.App

# Run with fat jar (simpler — no classpath needed)
mvn clean package -P fatjar
java -jar target/word-frequency-analyzer-1.0-SNAPSHOT.jar

# Run with a minimum-frequency threshold of 3
java -jar target/word-frequency-analyzer-1.0-SNAPSHOT.jar 3

# Show the dependency tree (transitive dependencies)
mvn dependency:tree

# Show effective POM (all inherited/default settings)
mvn help:effective-pom

# Skip tests during packaging
mvn package -DskipTests

# Run a single test class
mvn test -Dtest=WordFrequencyAnalyzerTest

# Clean build artifacts
mvn clean

# Run tests with coverage report (JaCoCo)
mvn clean verify
open target/site/jacoco/index.html   # View detailed coverage in browser
```

## Key Takeaways

| Without a Build System | With Maven |
|---|---|
| Manually download jars | Declare dependencies → auto-downloaded |
| `javac -cp lib1:lib2:... *.java` | `mvn compile` |
| Write your own test runner script | `mvn test` (surefire plugin) |
| Manually build jars with `jar` command | `mvn package` |
| "Works on my machine" | Reproducible builds everywhere |
| No standard project structure | Convention over configuration |
