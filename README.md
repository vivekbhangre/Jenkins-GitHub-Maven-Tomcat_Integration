# 🚀 DevOps CI/CD Pipeline — Jenkins + Maven + GitHub + Tomcat

> An end-to-end automated CI/CD pipeline that takes a Java web application from a GitHub push to a live deployment on Apache Tomcat — with **zero manual steps** in between.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Pipeline Flow](#-pipeline-flow)
- [Prerequisites](#-prerequisites)
- [Setup & Installation](#-setup--installation)
  - [1. Jenkins Setup](#1-jenkins-setup)
  - [2. Maven Setup](#2-maven-setup)
  - [3. GitHub Webhook](#3-github-webhook)
  - [4. Tomcat Setup](#4-tomcat-setup)
  - [5. Jenkins Freestyle Job Configuration](#5-jenkins-freestyle-job-configuration)
- [pom.xml](#-pomxml)
- [Running the Pipeline](#-running-the-pipeline)
- [Screenshots](#-screenshots)
- [Common Errors & Fixes](#-common-errors--fixes)
- [What I Learned](#-what-i-learned)
- [Future Improvements](#-future-improvements)

---

## 📌 Project Overview

This project demonstrates a real-world **Continuous Integration and Continuous Deployment (CI/CD)** workflow for a Java web application. It automates the entire software delivery lifecycle using a **Jenkins Freestyle Job** — no Jenkinsfile required.

| Stage | Tool | What Happens |
|-------|------|-------------|
| Source Control | GitHub | Code is stored and versioned |
| Trigger | GitHub Webhook | Push event fires Jenkins build automatically |
| Build & Test | Jenkins + Maven | Code compiled, tests run, WAR packaged |
| Deploy | Jenkins + Tomcat | WAR deployed to running Tomcat server |

The application is a Java JSP web app built using the standard Maven WAR project structure and deployed to Apache Tomcat 9.

---

## 🏗️ Architecture

```
Developer
    │
    │  git push origin main
    ▼
┌─────────────┐
│   GitHub    │  ──── Webhook (HTTP POST) ────►  ┌──────────────────────┐
│ Repository  │                                   │   Jenkins Freestyle  │
└─────────────┘                                   │         Job          │
                                                  │                      │
                                                  │  1. Git Checkout     │
                                                  │  2. mvn clean pkg    │
                                                  │  3. Deploy WAR       │
                                                  └──────────┬───────────┘
                                                             │
                                                             │  Deploy WAR via
                                                             │  Deploy Plugin
                                                             ▼
                                                  ┌──────────────────────┐
                                                  │   Apache Tomcat 9    │
                                                  │  localhost:8090/app  │
                                                  └──────────────────────┘
```

---

## 🛠️ Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **Jenkins** | 2.x LTS | CI/CD Automation Server (Freestyle Job) |
| **Apache Maven** | 3.9.x | Build tool & dependency management |
| **GitHub** | — | Source code repository + webhook trigger |
| **Apache Tomcat** | 9.x | Java web application server |
| **Java (JDK)** | 17 | Programming language & runtime |
| **JSP / Servlets** | 4.0 | Java web technology |

---

## 📁 Project Structure

```
devops-webapp/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/devops/
│   │   │       └── HelloServlet.java       # Sample servlet
│   │   └── webapp/
│   │       ├── WEB-INF/
│   │       │   └── web.xml                 # Deployment descriptor
│   │       └── index.jsp                   # Main web page (UI)
│   │
│   └── test/
│       └── java/
│           └── com/devops/
│               └── HelloServletTest.java   # JUnit test
│
├── pom.xml                                 # Maven project configuration
└── README.md                               # This file
```

> ℹ️ This project uses a **Jenkins Freestyle Job** — there is no Jenkinsfile. All pipeline configuration is done directly inside the Jenkins dashboard UI.

---

## 🔄 Pipeline Flow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  GIT CLONE   │───►│    BUILD     │───►│   PACKAGE    │───►│    DEPLOY    │
│              │    │              │    │              │    │              │
│ git checkout │    │ mvn compile  │    │  mvn package │    │  Upload WAR  │
│ from GitHub  │    │   + test     │    │  WAR created │    │  to Tomcat   │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
      ~3s                ~12s                ~5s                  ~5s
                                                      Total: ~25–40 seconds
```

**Each stage explained:**

1. **Git Checkout** — Jenkins pulls the latest code from the configured GitHub repository branch using stored credentials.

2. **Build** — Maven compiles all `.java` source files. Resolves and downloads all dependencies declared in `pom.xml` from Maven Central. JUnit tests run automatically during this phase.

3. **Package** — Maven bundles the compiled classes, JSPs, and `web.xml` into a `.war` (Web Application Archive) file inside the `target/` directory.

4. **Deploy** — The Jenkins **Deploy to Container Plugin** uploads the `.war` file to the Tomcat Manager API, which hot-deploys the application without restarting the server.

---

## ✅ Prerequisites

Before setting up this project, ensure the following are installed:

- [ ] **Java JDK 17+** — verify with `java -version`
- [ ] **Apache Maven 3.6+** — verify with `mvn -version`
- [ ] **Jenkins 2.x LTS** — running on `http://localhost:8080`
- [ ] **Apache Tomcat 9.x** — running on `http://localhost:8090`
- [ ] **Git** — verify with `git --version`
- [ ] A **GitHub account** with this repository

### Jenkins Plugins Required

Install from **Manage Jenkins → Plugin Manager → Available**:

| Plugin | Purpose |
|--------|---------|
| `Git Plugin` | Connects Jenkins to GitHub repo |
| `Maven Integration Plugin` | Adds Maven build step to Freestyle job |
| `Deploy to Container Plugin` | Deploys WAR to Tomcat after build |
| `GitHub Plugin` | Enables webhook trigger from GitHub |

---

## ⚙️ Setup & Installation

### 1. Jenkins Setup

```bash
# Start Jenkins (if running as JAR)
java -jar jenkins.war --httpPort=8080

# Or start as a service (Linux)
sudo systemctl start jenkins
```

**Configure JDK & Maven in Jenkins:**

1. Go to **Manage Jenkins → Global Tool Configuration**
2. Under **JDK** → click *Add JDK* → name it `JDK17` → set your `JAVA_HOME` path
3. Under **Maven** → click *Add Maven* → name it `Maven3` → tick *Install automatically* or provide the path
4. Click **Save**

---

### 2. Maven Setup

Verify Maven works locally before connecting to Jenkins:

```bash
mvn -version
# Apache Maven 3.9.x  /  Java version: 17.x

# Test build the project locally
mvn clean package

# Expected output:
# [INFO] BUILD SUCCESS
# target/devops-webapp.war  ← this file gets deployed to Tomcat
```

---

### 3. GitHub Webhook

This makes Jenkins trigger automatically on every `git push`:

1. Go to your GitHub repo → **Settings → Webhooks → Add webhook**
2. **Payload URL:**
   ```
   http://<your-jenkins-ip>:8080/github-webhook/
   ```
3. **Content type:** `application/json`
4. **Trigger:** Select *Just the push event*
5. Click **Add webhook** — GitHub will send a ping request to verify

> **Local setup tip:** If Jenkins is running on your local machine, GitHub can't reach it directly. Use [ngrok](https://ngrok.com) to create a public tunnel:
> ```bash
> ngrok http 8080
> # Use the generated https://xxxx.ngrok.io/github-webhook/ as your payload URL
> ```

---

### 4. Tomcat Setup

```bash
# Start Tomcat
cd $CATALINA_HOME/bin
./startup.sh        # Linux / Mac
startup.bat         # Windows

# Verify it's running at:
# http://localhost:8090
```

**Add a Manager user** — edit `$CATALINA_HOME/conf/tomcat-users.xml`:

```xml
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="admin"
        password="admin123"
        roles="manager-gui,manager-script"/>
</tomcat-users>
```

Restart Tomcat after saving. Verify login at `http://localhost:8090/manager/html`.

---

### 5. Jenkins Freestyle Job Configuration

**Step-by-step inside the Jenkins UI:**

#### 📁 General
- New Item → **Freestyle Project** → name it `devops-webapp` → OK

#### 🔗 Source Code Management
- Select **Git**
- Repository URL: `https://github.com/<your-username>/devops-webapp.git`
- Credentials: Add your GitHub username + personal access token
- Branch: `*/main`

#### ⚡ Build Triggers
- Check ✅ **GitHub hook trigger for GITScm polling**

#### 🔨 Build Environment
- *(optional)* Check *Delete workspace before build starts* for clean builds

#### 🏗️ Build Steps
- Click **Add build step** → *Invoke top-level Maven targets*
- Maven version: `Maven3`
- Goals: `clean package`

#### 📦 Post-build Actions
- Click **Add post-build action** → *Deploy WAR/EAR to a container*
- WAR/EAR files: `**/*.war`
- Context path: `/devops-webapp`
- Click **Add Container** → **Tomcat 9.x Remote**
  - Manager URL: `http://localhost:8090`
  - Credentials: Add Tomcat username (`admin`) and password (`admin123`)

Click **Save**.

---

## 📦 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.devops</groupId>
    <artifactId>devops-webapp</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>DevOps CI/CD Web Application</name>
    <description>Java web app demonstrating Jenkins Freestyle + Maven + GitHub + Tomcat pipeline</description>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <!-- Servlet API — provided by Tomcat at runtime -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>

        <!-- JSP API — provided by Tomcat at runtime -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
            <scope>provided</scope>
        </dependency>

        <!-- JUnit for unit testing -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <finalName>devops-webapp</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## ▶️ Running the Pipeline

### Automatic trigger (via webhook):
```bash
# Any push to main automatically starts the Jenkins build
git add .
git commit -m "your commit message"
git push origin main
# Jenkins receives the webhook → job starts within ~3 seconds
```

### Manual trigger:
1. Open Jenkins → click your `devops-webapp` job
2. Click **Build Now** on the left sidebar
3. Click the build number under *Build History* → **Console Output** to watch live logs

### Access the deployed application:
```
http://localhost:8090/devops-webapp
```

---

## 📸 Screenshots

> *(Replace with your actual screenshots after setup)*

| Screenshot | Description |
|-----------|-------------|
| `jenkins-job.png` | Jenkins Freestyle job showing green build status |
| `console-output.png` | Console output with `BUILD SUCCESS` and deploy confirmation |
| `tomcat-manager.png` | Tomcat Manager showing app listed as running |
| `github-webhook.png` | GitHub webhook settings with green tick (delivery success) |
| `live-app.png` | The deployed `index.jsp` page live in the browser |

---

## 🐛 Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `403 Forbidden` on Tomcat deploy | Manager user missing `manager-script` role | Update `tomcat-users.xml`, restart Tomcat |
| `GitHub webhook 302 redirect` | Wrong Jenkins URL format | Use exact URL with trailing `/github-webhook/` |
| `mvn: command not found` in Jenkins | Maven not configured in Global Tools | Set Maven path in *Global Tool Configuration* |
| `Port 8080 conflict` | Jenkins and Tomcat on same port | Change Tomcat to port `8090` in `server.xml` |
| `WAR not found` in post-build step | Wrong WAR path pattern | Use `**/*.war` instead of a hardcoded path |
| `Connection refused` to Tomcat | Tomcat not running | Run `./startup.sh` in `$CATALINA_HOME/bin` |
| Build passes but app not updated | Tomcat serving old cached version | Undeploy old WAR from Manager before deploying |

---

## 💡 What I Learned

- How to configure a **Jenkins Freestyle Job** end-to-end without writing any pipeline code
- How **Maven's build lifecycle** (`compile → test → package`) works and how each phase connects
- How **GitHub webhooks** send HTTP POST events to trigger external services
- The core difference between **CI** (Continuous Integration) and **CD** (Continuous Deployment)
- How **WAR files** are structured and how Tomcat's Manager API handles remote hot-deployments
- How to manage **credentials securely** in Jenkins using the built-in Credentials Store
- Debugging failed builds by reading **Jenkins console output** step by step

---

## 🔮 Future Improvements

- [ ] Migrate to a **Jenkins Pipeline Job** with a `Jenkinsfile` for version-controlled pipeline config
- [ ] Add **SonarQube** for static code analysis and quality gates
- [ ] Containerize the app with **Docker** and push image to Docker Hub
- [ ] Deploy to **Kubernetes** cluster
- [ ] Add **Slack or email notifications** on build success/failure
- [ ] Set up **multi-branch pipeline** for dev/staging/production environments
- [ ] Add **code coverage** reports using the JaCoCo Maven plugin

---
---

## 🙋‍♂️ Author

- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [your-linkedin](https://linkedin.com/in/your-profile)

---

> ⭐ If this project helped you understand DevOps concepts, consider giving it a star!
