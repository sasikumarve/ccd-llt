# 🚀 Containers and Cloud DevOps - LLT (CCD LLT)

This project demonstrates a secure, automated CI/CD pipeline that builds and deploys Docker images to AWS ECR using GitHub Actions and AWS OpenID Connect (OIDC) authentication.

---

## 🔁 Pipeline Overview

The GitHub Actions pipeline is triggered on every push to the `main` branch and follows these steps:

1. ✅ **Checkout code**  
2. 🔐 **Configure AWS credentials using OIDC**
3. 🗃️ **Ensure the target ECR repository exists**
4. 🔑 **Authenticate Docker with Amazon ECR**
5. 🛠️ **Build Docker image using a multi-stage build**
6. 🏷️ **Tag the Docker image**
7. 📤 **Push the Docker image to Amazon ECR**

---

## 🔐 IAM Role Design for GitHub OIDC

To enable secure access from GitHub to AWS, an IAM Role is created with the following trust policy conditions:

- **OIDC Provider**: `token.actions.githubusercontent.com`
- **Audience**: `sts.amazonaws.com`
- **Repository Restriction**:  
  Allows only a specific GitHub organization and repository (with optional branch) to assume the role via:

  ```json
  "Condition": {
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:<your-org>/<your-repo>:*"
    }
  }

---

## ⚠️ Challenges Faced

During the implementation of the CI/CD pipeline and Docker-based deployment, the following considerations and learning moments were noted:

### ✅ GitHub OIDC Integration with AWS

- **Challenge**: Understanding how GitHub’s OpenID Connect (OIDC) integrates with AWS IAM roles was initially complex.
- **Resolution**: Successfully set up a trust policy to allow secure authentication using GitHub OIDC without static AWS credentials.

### ✅ ECR Repository Not Visible

- **Challenge**: Image push appeared successful in GitHub Actions logs, but the image was not visible in the ECR console.
- **Resolution**: Added a step to explicitly create the repository if it doesn't exist, and used unique tags (e.g., `${{ github.sha }}`) to ensure pushes are not silently skipped.

### 🐳 Docker Image Optimization

- **Challenge**: Ensuring the final Docker image was optimized and minimal.
- **Resolution**: Implemented a multi-stage Docker build to separate the Maven build process from the runtime container, significantly reducing image size.

> Overall, no blockers were encountered, but attention to detail in IAM policies, tagging, and repository setup was key to successful deployment.

---

## 🐳 Docker Multi-Stage Build

This project uses a **multi-stage Docker build** to optimize the size and security of the final Docker image. The first stage compiles the Java code using Maven, and the second stage packages the compiled JAR into a lightweight runtime image.

### 🔧 Build Stages

### 🐳 Docker Multi-Stage Build

This project uses a **multi-stage Docker build** to efficiently compile and package a Java application into a minimal runtime image.

```dockerfile
# 🧱 Stage 1: Build with Maven
FROM maven:3.9.4-eclipse-temurin-17 AS builder

WORKDIR /app
COPY . .

RUN mvn clean package -DskipTests

# 🚀 Stage 2: Run with Minimal JDK
FROM eclipse-temurin:17-jdk-alpine

WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]



