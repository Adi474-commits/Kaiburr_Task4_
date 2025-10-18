# Task 4: CI-CD Pipeline with Jenkins and Kubernetes


## Author

**Adithya N Reddy**
- Email: adithyasnr@gmail.com
- BL.EN.U4EAC22075
- Electronics and Computer Engineering
- Amrita School Of Engineering Bengaluru.


## Overview

This repository contains a complete CI-CD pipeline implementation for the Kaiburr Task Manager application using Jenkins, Docker, and Kubernetes. The pipeline automates the entire software delivery process from source code to production deployment.

##  Architecture

### Pipeline Flow
```
GitHub → Jenkins → Build & Test → Docker Build → Registry Push → Kubernetes Deploy
```

### Components
- **Source Control:** GitHub repository with webhook integration
- **CI-CD Tool:** Jenkins with Pipeline as Code
- **Containerization:** Docker with multi-stage builds
- **Container Registry:** Docker Hub (configurable)
- **Orchestration:** Kubernetes cluster
- **Monitoring:** Health checks and logging

## Project Structure

```
task4-cicd-pipeline/
├── Jenkinsfile                     # Pipeline configuration
├── docker/                         # Docker configurations
│   ├── Dockerfile.backend          # Backend container
│   ├── Dockerfile.frontend         # Frontend container
│   ├── nginx.conf                  # Nginx configuration
│   └── default.conf                # Nginx site configuration
├── k8s/                            # Kubernetes manifests
│   ├── backend-deployment.yaml     # Backend deployment & service
│   ├── frontend-deployment.yaml    # Frontend deployment & service
│   ├── mongodb-deployment.yaml     # MongoDB deployment & service
│   ├── rbac.yaml                   # RBAC and security policies
│   └── ingress.yaml                # Ingress configuration
├── jenkins/                        # Jenkins documentation
│   └── JENKINS_SETUP.md           # Setup instructions
├── screenshots/                    # Documentation screenshots
│   └── SCREENSHOT_REQUIREMENTS.md # Screenshot guide
└── README.md                       # This file
```

## Quick Start

### Prerequisites

- **Jenkins** (v2.400+) with required plugins
- **Docker** (v20.0+) and Docker Hub account
- **Kubernetes** cluster (v1.25+) with kubectl configured
- **Git** repository with webhook access
- **Java 17** and **Maven 3.9** for backend builds
- **Node.js 18** and **npm** for frontend builds

### Installation Steps

1. **Clone the repository:**
```bash
git clone https://github.com/your-username/kaiburr-task-manager.git
cd kaiburr-task-manager
```

2. **Setup Jenkins:**
```bash
# Follow detailed instructions in jenkins/JENKINS_SETUP.md
docker-compose up -d jenkins
```

3. **Configure credentials in Jenkins:**
   - GitHub personal access token
   - Docker Hub credentials
   - Kubernetes config file

4. **Create pipeline job:**
   - Use Jenkinsfile from `task4-cicd-pipeline/Jenkinsfile`
   - Configure GitHub webhook
   - Set build parameters

5. **Deploy to Kubernetes:**
```bash
kubectl apply -f task4-cicd-pipeline/k8s/
```

##  Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DOCKER_REGISTRY` | Docker registry URL | `docker.io` |
| `DOCKER_REPOSITORY` | Docker repository name | `your-username/kaiburr-task-manager` |
| `K8S_NAMESPACE` | Kubernetes namespace | `kaiburr-app` |
| `JAVA_VERSION` | Java version for builds | `17` |
| `NODE_VERSION` | Node.js version for builds | `18` |

### Jenkins Credentials

Required credentials in Jenkins:
- `dockerhub-credentials`: Docker Hub username/password
- `github-token`: GitHub personal access token
- `kubeconfig-credentials`: Kubernetes configuration file
- `mongodb-credentials`: MongoDB username/password

## Pipeline Stages

### 1. Checkout
- Clones source code from GitHub
- Sets build version and image tags
- Prepares workspace

### 2. Environment Setup
- Configures Java and Node.js environments
- Validates tool versions
- Sets up build dependencies

### 3. Backend Build & Test
- Compiles Java application with Maven
- Runs unit tests and generates reports
- Creates JAR artifact
- Archives build results

### 4. Frontend Build & Test
- Installs npm dependencies
- Runs TypeScript compilation
- Executes frontend tests
- Builds production assets

### 5. Code Quality Analysis
- Runs ESLint for frontend
- Executes Checkstyle for backend
- Performs SpotBugs analysis
- Generates quality reports

### 6. Security Scanning
- OWASP dependency check
- npm security audit
- Container vulnerability scanning
- Generates security reports

### 7. Docker Image Build
- Builds backend Docker image
- Builds frontend Docker image
- Uses multi-stage builds for optimization
- Tags images with build version

### 8. Docker Image Security Scan
- Scans images with Trivy
- Checks for vulnerabilities
- Generates security reports
- Fails on critical issues

### 9. Push to Registry
- Authenticates with Docker registry
- Pushes versioned images
- Tags latest on main branch
- Updates image manifests

### 10. Deploy to Kubernetes
- Updates deployment manifests
- Applies Kubernetes resources
- Waits for rollout completion
- Verifies deployment health

### 11. Integration Tests
- Tests API endpoints
- Verifies frontend accessibility
- Checks service connectivity
- Validates deployment success

### 12. Performance Tests
- Runs load testing (main branch)
- Measures response times
- Generates performance reports
- Sets performance baselines

##  Docker Configuration

### Backend Dockerfile
- **Base Image:** OpenJDK 17 JRE Slim
- **Security:** Non-root user, minimal packages
- **Optimization:** Multi-stage build, layer caching
- **Health Checks:** Spring Boot Actuator endpoints

### Frontend Dockerfile
- **Base Image:** Node.js 18 Alpine for build, Nginx Alpine for runtime
- **Security:** Non-root user, security headers
- **Optimization:** Static asset compression, caching
- **Health Checks:** Nginx health endpoint

##  Kubernetes Deployment

### Resources Created
- **Namespace:** `kaiburr-app`
- **Deployments:** backend, frontend, mongodb
- **Services:** ClusterIP and NodePort services
- **ConfigMaps:** Application and database configuration
- **Secrets:** Database credentials and certificates
- **RBAC:** Service accounts and role bindings
- **Ingress:** External access configuration

### Security Features
- Pod Security Policies
- Network Policies
- Service Account permissions
- Secret management
- Container security contexts

## Monitoring and Observability

### Health Checks
- **Backend:** `/actuator/health` endpoint
- **Frontend:** `/health` endpoint
- **Database:** MongoDB ping command

### Logging
- Centralized logging with structured format
- Application and infrastructure logs
- Error tracking and alerting

### Metrics
- Prometheus-compatible metrics
- Application performance monitoring
- Infrastructure resource monitoring


##  Screenshots

The following screenshots document the complete CI-CD pipeline implementation:

 1. Project Directory Structure
    
<img width="1063" height="222" alt="Adithya N Reddy 17102025 adithyasnr@gmail com" src="https://github.com/user-attachments/assets/514576e6-e722-49db-97c0-aa0550cfb7a9" />


 2. Jenkins Setup with Docker Compose
    

<img width="527" height="227" alt="Task_4_SS2" src="https://github.com/user-attachments/assets/561902a9-5cb3-4232-897e-cacc3a2028a1" />

<img width="476" height="188" alt="Screenshot 2025-10-18 112942" src="https://github.com/user-attachments/assets/3fae9fae-af09-4aad-8ea9-0dce6c44ce43" />

 3. Kubernetes Manifests Overview
    

<img width="635" height="353" alt="Task_4_SS3" src="https://github.com/user-attachments/assets/244a31b9-d42f-4575-ae5c-5003735f6506" />

<img width="476" height="188" alt="Screenshot 2025-10-18 112942" src="https://github.com/user-attachments/assets/3fae9fae-af09-4aad-8ea9-0dce6c44ce43" />

 4. Jenkins Setup Documentation
    

<img width="623" height="95" alt="Task_4_SS4" src="https://github.com/user-attachments/assets/84109f5c-e4a4-4f13-bdca-e2871a9e6eb5" />

<img width="476" height="188" alt="Screenshot 2025-10-18 112942" src="https://github.com/user-attachments/assets/3fae9fae-af09-4aad-8ea9-0dce6c44ce43" />

 5. CI/CD Pipeline Features
    

<img width="638" height="83" alt="Task_4_SS5" src="https://github.com/user-attachments/assets/265d4357-1b83-4e94-bf2d-ef0bb6819baa" />

<img width="476" height="188" alt="Screenshot 2025-10-18 112942" src="https://github.com/user-attachments/assets/3fae9fae-af09-4aad-8ea9-0dce6c44ce43" />

 6. Application Dockerization Features
    

<img width="633" height="130" alt="Task_4_SS6" src="https://github.com/user-attachments/assets/8efeb7e4-75ec-4b8f-9677-e1911adc4e7b" />

<img width="476" height="188" alt="Screenshot 2025-10-18 112942" src="https://github.com/user-attachments/assets/3fae9fae-af09-4aad-8ea9-0dce6c44ce43" />

 7. Production-Ready Kubernetes Features
    

<img width="641" height="149" alt="Task_4_SS7" src="https://github.com/user-attachments/assets/e92c24ef-9eba-4d1f-9a19-83094bf27f97" />

<img width="476" height="188" alt="Screenshot 2025-10-18 112942" src="https://github.com/user-attachments/assets/3fae9fae-af09-4aad-8ea9-0dce6c44ce43" />


## Troubleshooting

### Common Issues

#### Build Failures
```bash
# Check Java version
java -version

# Verify Maven configuration
mvn --version

# Clear Maven cache
rm -rf ~/.m2/repository

# Check Node.js setup
node --version
npm --version
```

#### Docker Issues
```bash
# Check Docker daemon
docker version

# Login to registry
docker login

# Clean up images
docker system prune -f
```

#### Kubernetes Problems
```bash
# Check cluster connectivity
kubectl cluster-info

# Verify namespace
kubectl get namespace kaiburr-app

# Check pod status
kubectl get pods -n kaiburr-app

# View pod logs
kubectl logs -n kaiburr-app deployment/kaiburr-backend
```

### Debug Commands

#### Pipeline Debug
```groovy
// Add to Jenkinsfile for debugging
stage('Debug Environment') {
    steps {
        sh 'printenv | sort'
        sh 'docker version'
        sh 'kubectl version --client'
    }
}
```

#### Application Debug
```bash
# Test API endpoints
curl -f http://localhost:30088/actuator/health

# Test frontend
curl -f http://localhost:30080/health

# Check database connection
kubectl exec -it deployment/mongodb -n kaiburr-app -- mongosh
```






