# Onboarding Log / DevEx Audit

**Project:** Spring PetClinic
**Date:** February 2025
**Environment:** macOS (Apple Silicon) (Pablo Pineda), Windows (Windows 11) (Christian Martinez)

## 1. Repository Setup
Forked and cloned the official repository. Standard Git workflow was successful; no permission or access issues were encountered during the initial fetch.

## 2. Environment Verification & Friction Points
Upon the first attempt to execute the application using the wrapper (`./mvnw spring-boot:run`), the build failed immediately.

* **Issue:** The host operating system did not have a Java Development Kit (JDK) pre-installed.
* **Observation:** The error message indicated a missing `JAVA_HOME`. While the documentation states "Java 17 or later" is required, it assumes a pre-configured development environment.
* **Resolution Strategy:** Instead of installing the minimum requirement (LTS 17), I decided to install **OpenJDK 25** manually to test the application's forward compatibility with future JVM releases.

## 3. Runtime Compatibility
After configuring the JDK path, the application was executed again.

* **Result:** Build and startup were successful.
* **Performance:** Application started.

## 4. Database Configuration Findings
The `README.md` heavily implies the need for Docker to set up MySQL or PostgreSQL before running the app. However, runtime analysis of the logs revealed that the application defaults to an **H2 In-Memory Database** if no specific profile is active.

* **Impact:** This fallback allows for immediate execution without infrastructure overhead ("Zero-Config" start).
* **Critique:** The documentation could be clearer about this capability to prevent new developers from spending unnecessary time troubleshooting Docker containers for a simple first run.