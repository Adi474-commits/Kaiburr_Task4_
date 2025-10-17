# Task 4: CI-CD Pipeline with Jenkins and Kubernetes

## Overview

This repository contains a complete CI-CD pipeline implementation for the Kaiburr Task Manager application using Jenkins, Docker, and Kubernetes. The pipeline automates the entire software delivery process from source code to production deployment.

## 🏗️ Architecture

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

## 📁 Project Structure

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

## 🚀 Quick Start

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

## 🔧 Configuration

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

## 🔄 Pipeline Stages

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

## 🐳 Docker Configuration

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

## ☸️ Kubernetes Deployment

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

## 📊 Monitoring and Observability

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

## 🔒 Security

### Container Security
- Non-root user execution
- Minimal base images
- Vulnerability scanning
- Security context restrictions

### Kubernetes Security
- RBAC permissions
- Network policies
- Pod security policies
- Secret encryption

### Pipeline Security
- Credential management
- Build isolation
- Security scanning
- Audit logging

## 📸 Screenshots

The following screenshots document the complete CI-CD pipeline implementation:

1. **Jenkins Dashboard** - Main Jenkins interface and job overview
2. **Pipeline Configuration** - Job setup and parameter configuration
3. **Credentials Management** - Security credential setup
4. **Build Stage Execution** - Live pipeline execution view
5. **Test Results** - Unit test reports and coverage
6. **Docker Build Process** - Container image creation
7. **Registry Push** - Image deployment to registry
8. **Kubernetes Deployment** - Application deployment process
9. **Cluster Status** - Running pods and services
10. **Health Checks** - Application health verification
11. **Running Application** - Live application interface
12. **Pipeline Success** - Complete execution overview

*See `screenshots/` directory for detailed images and `SCREENSHOT_REQUIREMENTS.md` for capture guidelines.*

## 🚨 Troubleshooting

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

## 📈 Performance

### Build Optimization
- Parallel stage execution
- Docker layer caching
- Maven dependency caching
- npm package caching

### Resource Requirements
- **Jenkins Agent:** 2 CPU, 4GB RAM
- **Backend Pod:** 0.5 CPU, 512MB RAM
- **Frontend Pod:** 0.1 CPU, 128MB RAM
- **Database Pod:** 0.5 CPU, 512MB RAM

## 🔄 Continuous Improvement

### Metrics to Monitor
- Build success rate
- Build duration
- Deployment frequency
- Mean time to recovery
- Test coverage

### Optimization Opportunities
- Parallel test execution
- Advanced caching strategies
- Blue-green deployments
- Canary releases
- Auto-scaling configuration

## 📚 Additional Resources

### Documentation
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

### Tools and Plugins
- [Blue Ocean](https://www.jenkins.io/projects/blueocean/) - Modern Jenkins UI
- [Trivy](https://github.com/aquasecurity/trivy) - Vulnerability scanner
- [SonarQube](https://www.sonarqube.org/) - Code quality analysis

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes and test thoroughly
4. Submit a pull request
5. Update documentation as needed

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👨‍💻 Author

**Adithya N Reddy**
- GitHub: [@your-username](https://github.com/your-username)
- Email: your.email@example.com

## 🏷️ Version History

- **v1.0.0** - Initial CI-CD pipeline implementation
- **v1.1.0** - Added security scanning and optimization
- **v1.2.0** - Enhanced Kubernetes deployment and monitoring

---

**Note:** This CI-CD pipeline demonstrates enterprise-grade DevOps practices including automated testing, security scanning, containerization, and orchestrated deployment. It serves as a complete example of modern software delivery automation.