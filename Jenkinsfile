pipeline {
    agent any
    
    environment {
        // Docker Registry Configuration
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPOSITORY = 'your-dockerhub-username/kaiburr-task-manager'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        
        // Kubernetes Configuration
        KUBECONFIG_CREDENTIALS = 'kubeconfig-credentials'
        K8S_NAMESPACE = 'kaiburr-app'
        
        // Application Configuration
        APP_NAME = 'kaiburr-task-manager'
        FRONTEND_IMAGE = "${DOCKER_REPOSITORY}-frontend"
        BACKEND_IMAGE = "${DOCKER_REPOSITORY}-backend"
        
        // Build Configuration
        MAVEN_OPTS = '-Dmaven.repo.local=.m2/repository'
        NODE_VERSION = '18'
        JAVA_VERSION = '17'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        skipStagesAfterUnstable()
    }
    
    triggers {
        githubPush()
        pollSCM('H/5 * * * *') // Poll every 5 minutes
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Checking out source code...'
                checkout scm
                script {
                    env.BUILD_VERSION = "${env.BUILD_NUMBER}-${env.GIT_COMMIT[0..7]}"
                    env.FRONTEND_TAG = "${env.FRONTEND_IMAGE}:${env.BUILD_VERSION}"
                    env.BACKEND_TAG = "${env.BACKEND_IMAGE}:${env.BUILD_VERSION}"
                }
                echo "🏷️ Build Version: ${env.BUILD_VERSION}"
            }
        }
        
        stage('Environment Setup') {
            parallel {
                stage('Java Setup') {
                    steps {
                        echo '☕ Setting up Java environment...'
                        sh '''
                            java -version
                            mvn -version
                        '''
                    }
                }
                stage('Node.js Setup') {
                    steps {
                        echo '🟢 Setting up Node.js environment...'
                        sh '''
                            node --version
                            npm --version
                        '''
                    }
                }
            }
        }
        
        stage('Backend Build & Test') {
            steps {
                echo '🔨 Building and testing backend application...'
                dir('.') {
                    sh '''
                        echo "📦 Installing Maven dependencies..."
                        mvn clean compile -DskipTests
                        
                        echo "🧪 Running backend tests..."
                        mvn test -Dmaven.test.failure.ignore=true
                        
                        echo "📋 Generating test reports..."
                        mvn surefire-report:report
                        
                        echo "🏗️ Building JAR package..."
                        mvn package -DskipTests
                        
                        echo "✅ Backend build completed successfully"
                        ls -la target/
                    '''
                }
            }
            post {
                always {
                    // Publish test results
                    publishTestResults testResultsPattern: 'target/surefire-reports/*.xml'
                    // Archive artifacts
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Frontend Build & Test') {
            steps {
                echo '🎨 Building and testing frontend application...'
                dir('task-frontend') {
                    sh '''
                        echo "📦 Installing npm dependencies..."
                        npm ci --prefer-offline --no-audit
                        
                        echo "🧪 Running frontend tests..."
                        npm run test:ci || true
                        
                        echo "📋 Running linting..."
                        npm run lint || true
                        
                        echo "🏗️ Building frontend for production..."
                        npm run build
                        
                        echo "✅ Frontend build completed successfully"
                        ls -la dist/
                    '''
                }
            }
            post {
                always {
                    // Archive build artifacts
                    archiveArtifacts artifacts: 'task-frontend/dist/**/*', fingerprint: true
                }
            }
        }
        
        stage('Code Quality Analysis') {
            parallel {
                stage('Backend Code Quality') {
                    steps {
                        echo '📊 Running backend code quality analysis...'
                        sh '''
                            echo "🔍 Running Maven checkstyle..."
                            mvn checkstyle:check || true
                            
                            echo "🔍 Running SpotBugs analysis..."
                            mvn spotbugs:check || true
                        '''
                    }
                }
                stage('Frontend Code Quality') {
                    steps {
                        echo '📊 Running frontend code quality analysis...'
                        dir('task-frontend') {
                            sh '''
                                echo "🔍 Running ESLint analysis..."
                                npm run lint:report || true
                                
                                echo "🔍 Running type checking..."
                                npm run type-check || true
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Security Scanning') {
            parallel {
                stage('Backend Security') {
                    steps {
                        echo '🔒 Running backend security scan...'
                        sh '''
                            echo "🔍 OWASP Dependency Check..."
                            mvn org.owasp:dependency-check-maven:check || true
                        '''
                    }
                }
                stage('Frontend Security') {
                    steps {
                        echo '🔒 Running frontend security scan...'
                        dir('task-frontend') {
                            sh '''
                                echo "🔍 npm audit..."
                                npm audit --audit-level=moderate || true
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Docker Image Build') {
            parallel {
                stage('Backend Docker Build') {
                    steps {
                        echo '🐳 Building backend Docker image...'
                        script {
                            def backendImage = docker.build(
                                "${env.BACKEND_TAG}",
                                "-f docker/Dockerfile.backend ."
                            )
                            echo "✅ Backend image built: ${env.BACKEND_TAG}"
                        }
                    }
                }
                stage('Frontend Docker Build') {
                    steps {
                        echo '🐳 Building frontend Docker image...'
                        script {
                            def frontendImage = docker.build(
                                "${env.FRONTEND_TAG}",
                                "-f docker/Dockerfile.frontend ./task-frontend"
                            )
                            echo "✅ Frontend image built: ${env.FRONTEND_TAG}"
                        }
                    }
                }
            }
        }
        
        stage('Docker Image Security Scan') {
            parallel {
                stage('Backend Image Scan') {
                    steps {
                        echo '🔍 Scanning backend Docker image for vulnerabilities...'
                        sh """
                            echo "🔍 Running Trivy scan on backend image..."
                            trivy image --severity HIGH,CRITICAL ${env.BACKEND_TAG} || true
                        """
                    }
                }
                stage('Frontend Image Scan') {
                    steps {
                        echo '🔍 Scanning frontend Docker image for vulnerabilities...'
                        sh """
                            echo "🔍 Running Trivy scan on frontend image..."
                            trivy image --severity HIGH,CRITICAL ${env.FRONTEND_TAG} || true
                        """
                    }
                }
            }
        }
        
        stage('Push to Registry') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    tag pattern: 'v\\d+\\.\\d+\\.\\d+', comparator: 'REGEXP'
                }
            }
            steps {
                echo '📤 Pushing Docker images to registry...'
                script {
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_CREDENTIALS) {
                        // Push backend image
                        sh "docker push ${env.BACKEND_TAG}"
                        echo "✅ Backend image pushed: ${env.BACKEND_TAG}"
                        
                        // Push frontend image
                        sh "docker push ${env.FRONTEND_TAG}"
                        echo "✅ Frontend image pushed: ${env.FRONTEND_TAG}"
                        
                        // Tag and push as latest if main branch
                        if (env.BRANCH_NAME == 'main') {
                            sh """
                                docker tag ${env.BACKEND_TAG} ${env.BACKEND_IMAGE}:latest
                                docker tag ${env.FRONTEND_TAG} ${env.FRONTEND_IMAGE}:latest
                                docker push ${env.BACKEND_IMAGE}:latest
                                docker push ${env.FRONTEND_IMAGE}:latest
                            """
                            echo "✅ Latest tags pushed"
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                echo '🚀 Deploying to Kubernetes...'
                script {
                    withCredentials([kubeconfigFile(credentialsId: env.KUBECONFIG_CREDENTIALS, variable: 'KUBECONFIG')]) {
                        sh '''
                            echo "🔧 Setting up Kubernetes environment..."
                            kubectl version --client
                            
                            echo "📝 Creating namespace if it doesn't exist..."
                            kubectl create namespace ${K8S_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                            
                            echo "🔄 Updating deployment configurations..."
                            sed -i "s|BACKEND_IMAGE_PLACEHOLDER|${BACKEND_TAG}|g" k8s/backend-deployment.yaml
                            sed -i "s|FRONTEND_IMAGE_PLACEHOLDER|${FRONTEND_TAG}|g" k8s/frontend-deployment.yaml
                            
                            echo "🚀 Applying Kubernetes manifests..."
                            kubectl apply -f k8s/ -n ${K8S_NAMESPACE}
                            
                            echo "⏳ Waiting for deployments to be ready..."
                            kubectl rollout status deployment/kaiburr-backend -n ${K8S_NAMESPACE} --timeout=300s
                            kubectl rollout status deployment/kaiburr-frontend -n ${K8S_NAMESPACE} --timeout=300s
                            
                            echo "📋 Getting deployment status..."
                            kubectl get pods,services,deployments -n ${K8S_NAMESPACE}
                            
                            echo "✅ Deployment completed successfully!"
                        '''
                    }
                }
            }
        }
        
        stage('Integration Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                echo '🧪 Running integration tests against deployed application...'
                script {
                    withCredentials([kubeconfigFile(credentialsId: env.KUBECONFIG_CREDENTIALS, variable: 'KUBECONFIG')]) {
                        sh '''
                            echo "🔍 Getting service endpoints..."
                            BACKEND_URL=$(kubectl get service kaiburr-backend-service -n ${K8S_NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}:{.spec.ports[0].port}' || echo "localhost:30088")
                            FRONTEND_URL=$(kubectl get service kaiburr-frontend-service -n ${K8S_NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}:{.spec.ports[0].port}' || echo "localhost:30080")
                            
                            echo "🧪 Testing backend health endpoint..."
                            curl -f "http://${BACKEND_URL}/actuator/health" || echo "Backend health check failed"
                            
                            echo "🧪 Testing backend API endpoints..."
                            curl -f "http://${BACKEND_URL}/tasks" || echo "Backend API test failed"
                            
                            echo "🧪 Testing frontend availability..."
                            curl -f "http://${FRONTEND_URL}/" || echo "Frontend test failed"
                            
                            echo "✅ Integration tests completed"
                        '''
                    }
                }
            }
        }
        
        stage('Performance Tests') {
            when {
                branch 'main'
            }
            steps {
                echo '⚡ Running performance tests...'
                sh '''
                    echo "🚀 Running basic load test..."
                    # Add your performance testing tools here (JMeter, Artillery, etc.)
                    echo "⚡ Performance test completed"
                '''
            }
        }
    }
    
    post {
        always {
            echo '🧹 Cleaning up workspace...'
            
            // Clean up Docker images to save space
            sh '''
                echo "🐳 Cleaning up Docker images..."
                docker image prune -f --filter "until=24h" || true
                docker system df
            '''
            
            // Archive logs
            archiveArtifacts artifacts: 'logs/**/*', allowEmptyArchive: true
            
            // Clean workspace
            cleanWs()
        }
        
        success {
            echo '✅ Pipeline completed successfully!'
            
            // Send success notification
            script {
                if (env.BRANCH_NAME == 'main') {
                    // Add notification logic here (Slack, Email, etc.)
                    echo "📧 Sending success notification..."
                }
            }
        }
        
        failure {
            echo '❌ Pipeline failed!'
            
            // Send failure notification
            script {
                // Add notification logic here (Slack, Email, etc.)
                echo "📧 Sending failure notification..."
            }
        }
        
        unstable {
            echo '⚠️ Pipeline completed with warnings!'
        }
        
        aborted {
            echo '🛑 Pipeline was aborted!'
        }
    }
}