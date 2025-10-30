# Jenkins Docker CI/CD Pipeline

Production-ready Jenkins CI/CD pipeline with Docker, Kubernetes, and comprehensive testing.

## 🚀 Features

- **Complete CI/CD Pipeline**: Build, test, security scan, deploy
- **Multi-Environment**: Staging and production deployments
- **Docker Integration**: Containerized builds and deployments
- **Kubernetes Ready**: Production K8s manifests
- **Security Scanning**: Trivy, SonarQube, npm audit
- **Performance Testing**: JMeter integration
- **Monitoring**: Health checks and metrics endpoints

## 📁 Project Structure

```
├── Jenkinsfile              # Pipeline definition
├── app/
│   ├── src/server.js        # Node.js application
│   ├── tests/               # Unit and integration tests
│   ├── Dockerfile           # Multi-stage Docker build
│   └── package.json         # Dependencies and scripts
├── k8s/                     # Kubernetes manifests
├── jenkins/                 # Jenkins configuration
└── docker-compose.yml       # Local development stack
```

## 🛠️ Quick Start

### Prerequisites
- Docker & Docker Compose
- Jenkins 2.400+
- Kubernetes cluster (for deployment)
- SonarQube (optional)

### Local Development
```bash
# Clone repository
git clone https://github.com/iammaksudul/jenkins-docker-cicd-pipeline.git
cd jenkins-docker-cicd-pipeline

# Start Jenkins and SonarQube
docker-compose up -d

# Access Jenkins: http://localhost:8080
# Access SonarQube: http://localhost:9000
```

### Application Testing
```bash
cd app

# Install dependencies
npm install

# Run tests
npm test

# Start application
npm start

# Access: http://localhost:3000
```

## 🔄 Pipeline Stages

### 1. **Checkout**
- Source code checkout
- Git commit information

### 2. **Install Dependencies**
- npm ci for clean install
- Dependency caching

### 3. **Code Quality** (Parallel)
- **Linting**: ESLint code analysis
- **Security Scan**: npm audit + Snyk
- **SonarQube**: Code quality metrics

### 4. **Testing** (Parallel)
- **Unit Tests**: Jest with coverage
- **Integration Tests**: API endpoint testing

### 5. **Build**
- Application build process
- Asset optimization

### 6. **Docker Build**
- Multi-stage Docker build
- Security scanning with Trivy
- Push to Docker registry

### 7. **Deploy to Staging**
- Kubernetes deployment
- Health checks
- Rollout verification

### 8. **Performance Testing**
- JMeter load testing
- Performance metrics

### 9. **Deploy to Production**
- Manual approval gate
- Blue-green deployment
- Health verification

## 🐳 Docker Configuration

### Multi-Stage Build
```dockerfile
FROM node:18-alpine AS builder
# Build stage

FROM node:18-alpine AS runtime
# Runtime stage with non-root user
```

### Security Features
- Non-root user execution
- Health checks
- Minimal attack surface
- Security scanning

## ☸️ Kubernetes Deployment

### Production Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-app-prod
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
```

### Features
- Rolling updates
- Resource limits
- Health probes
- Ingress configuration

## 🔧 Jenkins Configuration

### Required Plugins
- Docker Pipeline
- Kubernetes
- SonarQube Scanner
- Performance Plugin
- Slack Notification

### Credentials Setup
```bash
# Docker Hub credentials
DOCKER_CREDENTIALS

# Kubernetes config
KUBECONFIG

# SonarQube token
SONAR_TOKEN

# Slack webhook
SLACK_WEBHOOK
```

## 📊 Monitoring & Observability

### Health Endpoints
- `/health` - Application health
- `/metrics` - Performance metrics
- `/api/status` - Service status

### Monitoring Integration
- Prometheus metrics
- Grafana dashboards
- Alert management
- Log aggregation

## 🧪 Testing Strategy

### Unit Tests
```bash
npm run test:unit
```

### Integration Tests
```bash
npm run test:integration
```

### Performance Tests
```bash
# JMeter load testing
jmeter -n -t load-test.jmx
```

### Security Tests
```bash
# Container scanning
trivy image app:latest

# Dependency scanning
npm audit
snyk test
```

## 🚀 Production Deployment

### Environment Variables
```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
```

### Scaling
```bash
# Horizontal scaling
kubectl scale deployment devops-app-prod --replicas=5

# Vertical scaling
kubectl patch deployment devops-app-prod -p '{"spec":{"template":{"spec":{"containers":[{"name":"devops-app","resources":{"limits":{"memory":"512Mi","cpu":"500m"}}}]}}}}'
```

## 📈 Performance Optimization

- **Caching**: Redis integration
- **CDN**: Static asset delivery
- **Database**: Connection pooling
- **Monitoring**: APM integration

## 🔒 Security Best Practices

- Container security scanning
- Dependency vulnerability checks
- HTTPS enforcement
- Secret management
- Network policies

## 🤝 Contributing

1. Fork repository
2. Create feature branch
3. Add tests for new features
4. Ensure pipeline passes
5. Submit pull request

## 📄 License

MIT License - see LICENSE file for details.

## 👨‍💻 Author

**Alam** - DevOps Engineer
- GitHub: [@iammaksudul](https://github.com/iammaksudul)
- Email: kh.maksudul.alam.cse@gmail.com

---

*Enterprise-grade CI/CD pipeline with production-ready configurations.*
