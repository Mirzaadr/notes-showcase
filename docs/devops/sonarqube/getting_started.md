# SonarQube & SonarScanner Setup Guide

This guide covers setting up a SonarQube server via Docker and executing a local code analysis using the SonarScanner CLI.

## 1. Start the SonarQube Server

Run the SonarQube server as a Docker container. Use the following commands based on your operating system.

<!-- tabs:start -->
### **Windows (PowerShell)**
```powershell
docker run -d --name sonarqube `
  -p 9000:9000 `
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true `
  sonarqube:latest

```

### **Linux**

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:latest

```

### **macOS**

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:latest

```
<!-- tabs:end -->

> **Note:** Wait 2-3 minutes after running the command for the server to initialize. Access it at `http://localhost:9000` (Default: `admin` / `admin`).

---

## 2. Prepare Your Project

1. Log in to the SonarQube dashboard.
2. Click **Create Project Manually**.
3. Set your **Project Key** (e.g., `my-portfolio`).
4. Generate a **Project Token** when prompted and keep it safe.
5. Create a `sonar-project.properties` file in your project's root directory:

```properties
sonar.projectKey=my-portfolio
sonar.sources=.
sonar.host.url=http://localhost:9000
sonar.token=your_generated_token_here

```

---

## 3. Run the Scan

Use the following commands from the root directory of your project to trigger the analysis.

<!-- tabs:start -->
### **Windows (PowerShell)**

```powershell
docker run --rm `
  -v "${PWD}:/usr/src" `
  sonarsource/sonar-scanner-cli

```

### **Linux**

```bash
docker run --rm \
  -v "$(pwd):/usr/src" \
  sonarsource/sonar-scanner-cli

```

### **macOS**

```bash
docker run --rm \
  -v "$(pwd):/usr/src" \
  sonarsource/sonar-scanner-cli

```
<!-- tabs:end -->

---

## Troubleshooting Tips

* **Network Issues:** If running SonarQube on Docker and the scanner cannot connect, replace `localhost` in your properties file with `host.docker.internal`.
* **Permissions:** Ensure your user has read access to the project folder being mounted.
* **Language Support:** If you are using languages like Dart (for Flutter), ensure the corresponding plugin is installed on your SonarQube server instance.
