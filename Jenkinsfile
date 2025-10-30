pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO = 'iammaksudul/devops-app'
        KUBECONFIG = credentials('kubeconfig')
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        SONAR_TOKEN = credentials('sonar-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                    env.BUILD_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT_SHORT}"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "Installing Node.js dependencies..."
                    cd app
                    npm ci
                '''
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Lint') {
                    steps {
                        sh '''
                            cd app
                            npm run lint
                        '''
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        sh '''
                            cd app
                            npm audit --audit-level high
                            npm run security-check
                        '''
                    }
                }
                
                stage('SonarQube Analysis') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh '''
                                cd app
                                sonar-scanner \
                                  -Dsonar.projectKey=devops-app \
                                  -Dsonar.sources=src \
                                  -Dsonar.tests=tests \
                                  -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh '''
                            cd app
                            npm run test:unit
                        '''
                    }
                    post {
                        always {
                            publishTestResults testResultsPattern: 'app/test-results.xml'
                            publishCoverage adapters: [
                                istanbulCoberturaAdapter('app/coverage/cobertura-coverage.xml')
                            ]
                        }
                    }
                }
                
                stage('Integration Tests') {
                    steps {
                        sh '''
                            cd app
                            npm run test:integration
                        '''
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    echo "Building application..."
                    cd app
                    npm run build
                '''
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    def image = docker.build("${DOCKER_REPO}:${BUILD_TAG}", "./app")
                    
                    // Security scan
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image --exit-code 1 --severity HIGH,CRITICAL \
                        ${DOCKER_REPO}:${BUILD_TAG}
                    """
                    
                    // Push to registry
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                    echo "Deploying to staging environment..."
                    envsubst < k8s/deployment-staging.yaml | kubectl apply -f -
                    kubectl rollout status deployment/devops-app-staging -n staging
                '''
            }
        }
        
        stage('Performance Tests') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                    echo "Running performance tests..."
                    cd tests/performance
                    jmeter -n -t load-test.jmx -l results.jtl
                '''
            }
            post {
                always {
                    perfReport sourceDataFiles: 'tests/performance/results.jtl'
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def deploy = input(
                        message: 'Deploy to production?',
                        parameters: [
                            choice(choices: ['Deploy', 'Abort'], description: 'Choose action', name: 'ACTION')
                        ]
                    )
                    
                    if (deploy == 'Deploy') {
                        sh '''
                            echo "Deploying to production environment..."
                            envsubst < k8s/deployment-prod.yaml | kubectl apply -f -
                            kubectl rollout status deployment/devops-app-prod -n production
                        '''
                        
                        // Health check
                        sh '''
                            echo "Performing health check..."
                            sleep 30
                            curl -f http://devops-app-prod.production.svc.cluster.local/health
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ Pipeline succeeded for ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
        
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "❌ Pipeline failed for ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
