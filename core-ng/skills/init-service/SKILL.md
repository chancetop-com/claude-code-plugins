---
name: init-service
description: Initialize a new service in a core-ng project.
---

## Description

This skill helps you add a new service sub-project to a core-ng gradle multi-module project. It automates:

1. Reading existing project configuration
2. Gathering user preferences
3. Adding the service module to `settings.gradle.kts`
4. Creating the standard directory structure
5. Creating essential service files (Main.java, App class, properties, Dockerfile)
6. Updating the root `build.gradle.kts` with project configuration
7. Verifying the setup with Gradle

## Usage

When the user requests to initialize a new service, follow these steps:

### 0. Read Existing Configuration

First, read the existing configuration to understand the project structure:

- Read `settings.gradle.kts` to see existing services and check for duplicates
- Read `build.gradle.kts` to:
   - Extract the `coreNGVersion` variable (if defined)
   - Understand existing service configurations
   - Determine the core-ng groupId pattern (usually `core.framework`)

### 1. Gather Information

Ask the user for:

- **Subfolder**: The subfolder which the new service belongs to (e.g., `backend`, `frontend`). Offer "Root level (Recommended)" as the first option if other services are at root, or suggest a specific
  subfolder if that's the pattern.
- **Service name**: Should be provided by the user (e.g., `customer-service`, `user-service`, `order-service`)
- **Package name**: The base Java package. Suggest options based on existing patterns:
   - If existing services use `app.*` pattern, suggest `app.{service-name}` (Recommended)
   - Alternative: use the Gradle group id pattern (e.g., `org.playground.{service-name}`)

**Note**: Do NOT ask for core-ng groupId or version - extract these from the root `build.gradle.kts`

### 2. Update settings.gradle.kts

Add the new service module to the `settings.gradle.kts` file in the project root:

```kts
include("service-name")
```

or if in a subfolder:

```kts
include("subfolder:service-name")
```

### 3. Update Root build.gradle.kts

Add the project configuration block to the root `build.gradle.kts` (do NOT create a separate build.gradle for the service):

```kts
project("subfolder:service-name") {
    apply(plugin = "app")
    dependencies {
        implementation("core.framework:core-ng:${coreNGVersion}")
        testImplementation("core.framework:core-ng-test:${coreNGVersion}")
    }
}
```

Use the path format that matches what you added to `settings.gradle.kts` (with or without subfolder prefix).

### 4. Create Directory Structure

Create the complete directory structure with a single mkdir command:

```bash
mkdir -p {subfolder}/service-name/docker \
         {subfolder}/service-name/conf/dev/resources \
         {subfolder}/service-name/src/main/java/{package-path} \
         {subfolder}/service-name/src/main/resources \
         {subfolder}/service-name/src/test/java/{package-path} \
         {subfolder}/service-name/src/test/resources
```

Where `{package-path}` is the package name converted to a path (e.g., `app.customer` → `app/customer`).

### 5. Create Essential Files

Create the following files with appropriate content:

**a) App Class** (at `src/main/java/{package-path}/{ServiceName}App.java`):

```java
package

{package.name};

import core.framework.module.App;

public class {ServiceName}App extends

App {
    @Override
    protected void initialize () {
        loadProperties("app.properties");
    }
}
```

**b) Main.java** (at `src/main/java/Main.java`):

```java
import {package.name}.{ServiceName}App;

public class Main {
    public static void main(String[] args) {
        new {
            ServiceName
        } App().start();
    }
}
```

**c) app.properties** (at `src/main/resources/app.properties`):

```properties
# {Service Name} Configuration
```

**d) Dockerfile** (at `docker/Dockerfile`):

```dockerfile
FROM        wonder/jdk:24-alpine
LABEL       app={service-name}
RUN         addgroup --system app && adduser --system --no-create-home --ingroup app app && apk add --no-cache gcompat
USER        app
COPY        package/dependency      /opt/app
COPY        package/app             /opt/app
CMD         ["/opt/app/bin/{service-name}"]
```

**Naming conventions:**

- Convert `customer-service` to `CustomerService` for class names
- Keep kebab-case for service names in paths and Docker
- Use dot notation for packages (e.g., `app.customer`)

### 6. Verify Setup

After creating all files, verify the setup:

1. **Check Gradle recognizes the module:**
   ```bash
   ./gradlew :subfolder:service-name:tasks --quiet
   ```
   Should show available tasks including `run`, `build`, etc.

2. **Build the service:**
   ```bash
   ./gradlew :subfolder:service-name:build
   ```
   Should complete successfully with "BUILD SUCCESSFUL"

3. **Verify created files:**
   - The service module appears in `settings.gradle.kts`
   - Project configuration exists in root `build.gradle.kts`
   - All directories are created correctly
   - All essential files (Main.java, App class, properties, Dockerfile) exist

## Example

User: "set up a new customer-service"

Expected actions:

**0. Read existing configuration:**

- Read `settings.gradle.kts`: see `include("system-coupling-analyzer")`
- Read `build.gradle.kts`: extract `coreNGVersion = "9.3.1"` and see core-ng groupId is `core.framework`

**1. Gather information via AskUserQuestion:**

- Subfolder: User chooses "subfolder named `random`"
- Service name: `customer-service` (from user's request)
- Package name: User chooses "app.customer (Recommended)"

**2. Update settings.gradle.kts:**

   ```kts
   include("system-coupling-analyzer")
include("random:customer-service")
   ```

**3. Update root build.gradle.kts:**
Add after the existing project configuration:

   ```kts
   project("random:customer-service") {
    apply(plugin = "app")
    dependencies {
        implementation("core.framework:core-ng:${coreNGVersion}")
        testImplementation("core.framework:core-ng-test:${coreNGVersion}")
    }
}
   ```

**4. Create directory structure:**

   ```bash
   mkdir -p random/customer-service/docker \
            random/customer-service/conf/dev/resources \
            random/customer-service/src/main/java/app/customer \
            random/customer-service/src/main/resources \
            random/customer-service/src/test/java/app/customer \
            random/customer-service/src/test/resources
   ```

**5. Create essential files:**

`random/customer-service/src/main/java/Main.java`:

   ```java
   public class Main {
    public static void main(String[] args) {
        new app.customer.CustomerServiceApp().start();
    }
}
   ```

`random/customer-service/src/main/java/app/customer/CustomerServiceApp.java`:

   ```java
   package app.customer;

import core.framework.module.App;

public class CustomerServiceApp extends App {
    @Override
    protected void initialize() {
        loadProperties("app.properties");
    }
}
   ```

`random/customer-service/src/main/resources/app.properties`:

   ```properties
   # Customer Service Configuration
   ```

`random/customer-service/docker/Dockerfile`:

   ```dockerfile
   FROM amazoncorretto:25-alpine

   RUN addgroup -g 1000 app && adduser -D -u 1000 -G app app
   USER app
   WORKDIR /opt/app

   ADD --chown=app:app dependency/lib         /opt/app/lib
   ADD --chown=app:app app                    /opt/app

   ENTRYPOINT ["/opt/app/bin/customer-service"]
   ```

**6. Verify setup:**

   ```bash
   # Check Gradle recognizes the module
   ./gradlew :random:customer-service:tasks --quiet

   # Build the service
   ./gradlew :random:customer-service:build
   ```

Expected: "BUILD SUCCESSFUL" message

**7. Provide summary to user:**
Show the created structure, available commands, and confirm the service is ready for development.

## Notes

- **Always read existing configuration first** - Extract core-ng version and understand patterns before asking user questions
- **Check for duplicates** - Verify the service name doesn't already exist in `settings.gradle.kts`
- **Naming conventions:**
   - Service names: kebab-case (e.g., `customer-service`, `user-service`)
   - Class names: PascalCase (e.g., `CustomerServiceApp`, `UserServiceApp`)
   - Package names: lowercase with dots (e.g., `app.customer`, `app.user`)
- **Use AskUserQuestion tool** - Present options with recommendations based on existing patterns
- **Don't create separate build.gradle** - Always update the root `build.gradle.kts` file
- **Verify with Gradle** - Always run `./gradlew :path:to:service:build` to ensure the setup works
- **Provide clear summary** - Show the user what was created and available commands

## Common Gradle Commands

After setup, the user can run:

```bash
# Build the service
./gradlew :{path}:service-name:build

# Run the service
./gradlew :{path}:service-name:run

# Run tests
./gradlew :{path}:service-name:test

# Code quality checks
./gradlew :{path}:service-name:check

# Create Docker image
./gradlew :{path}:service-name:docker
```
