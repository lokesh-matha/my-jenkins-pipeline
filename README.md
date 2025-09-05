# Jenkins Docker Pipeline Project

A CI/CD pipeline that automatically builds, tests, and deploys a Python Flask application using Jenkins and Docker.


## Features
- 🚀 **Automated builds** on code push to GitHub
- 📦 **Docker containerization** with multi-stage builds
- 🔄 **Auto-deployment** to production/staging
- ✅ **Health checks** and monitoring

## Prerequisites
- Jenkins server (`v2.300+`)
- Docker installed (`v20.10+`)
- GitHub repository
- Docker Hub account

## Setup Guide

### 1. Jenkins Configuration
```bash
# Install required plugins
- Docker Pipeline
- GitHub Integration
- Blue Ocean (optional)
```

### 2. GitHub Webhook
1. Go to: `Settings > Webhooks`
2. Add payload URL:  
   `http://<JENKINS_IP>:8080/github-webhook/`
3. Set content type: `application/json`

### 3. Environment Variables
Create these in Jenkins (`Manage Jenkins > System`):
```env
DOCKER_HUB_CREDENTIALS=jenkins-dockerhub-creds
DEPLOYMENT_SERVER_IP=192.168.1.100
```

## Pipeline Workflow
1. **On Git Push** → Triggers Jenkins build
2. **Build** → Creates Docker image
3. **Test** → Runs pytest inside container
4. **Push** → Uploads image to Docker Hub
5. **Deploy** → Updates production container

## Local Development
```bash
# Build image
docker build -t my-app .

# Run locally
docker run -p 5000:5000 my-app

# Test endpoints
curl http://localhost:5000
```

## Project Structure
```
├── app/
│   ├── app.py            # Flask application
│   └── requirements.txt  # Python dependencies
├── Jenkinsfile           # Pipeline definition
├── Dockerfile            # Container setup
└── tests/                # Unit tests
```
## output
```
Go to the Jenkins server (or your local machine if you copy the JAR).

Run this command:
```bash

java -jar C:\ProgramData\Jenkins\.jenkins\workspace\simple-java-app\target\hello-1.0.jar
```

This will execute your Java app and print the output (for example: Hello, World! if that’s what the app does).

```
## Troubleshooting
| Error | Solution |
|-------|----------|
| `403 Forbidden` from webhook | Check Jenkins security settings |
| Docker build fails | Verify `requirements.txt` exists |
| Push to Docker Hub fails | Re-login with `docker login` |

## FAQ
**Q: How do I trigger a manual build?**  
A: Go to Jenkins dashboard → Select project → "Build Now"

**Q: Where are Docker images stored?**  
A: Pushed to Docker Hub: `lokeshmatha/my-app-image`

---

> ✨ **Pro Tip**: Use `docker system prune` weekly to clean up old images.

[![Jenkins Build Status](https://img.shields.io/jenkins/build?jobUrl=http%3A%2F%2Fjenkins.example.com%2Fjob%2Fmy-app-pipeline)](https://jenkins.example.com/job/my-app-pipeline)
