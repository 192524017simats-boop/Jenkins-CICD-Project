# Jenkins CI/CD Practical – Flask Application

This project demonstrates a local CI/CD pipeline using Jenkins and Docker.

## Files
- `app/app.py` – Flask sample application
- `tests/test_app.py` – automated tests
- `requirements.txt` – Python dependencies
- `Dockerfile` – test and runtime image stages
- `Jenkinsfile` – checkout, build, test, deploy, and health-check stages

## Requirements
- Docker Desktop running
- Jenkins running in Docker
- Jenkins container has Docker CLI and `/var/run/docker.sock` mounted
- GitHub repository containing these files

## Jenkins setup
1. Put all project files in a GitHub repository. Keep `Jenkinsfile` in the repository root.
2. In Jenkins select **New Item**, enter `Flask-CI-CD`, choose **Pipeline**, and click **OK**.
3. Under **Pipeline**, choose **Pipeline script from SCM**.
4. SCM: **Git**. Repository URL: your GitHub repository HTTPS URL.
5. Branch: `*/main` (use `*/master` if that is your default branch).
6. Script Path: `Jenkinsfile`. Save, then click **Build Now**.

The pipeline checks Git for new commits about every two minutes using `pollSCM`.

## Successful deployment
After a successful pipeline, open:
- App: http://localhost:8081
- Health check: http://localhost:8081/health

## Optional manual Docker test
From PowerShell in this folder:
```powershell
docker build --target runtime -t student-flask-cicd:latest .
docker run -d --name student-flask-manual -p 8081:5000 student-flask-cicd:latest
```
Stop it with:
```powershell
docker rm -f student-flask-manual
```

## GitHub webhook / pull requests
A GitHub webhook requires a Jenkins URL reachable from GitHub. `localhost:8080` is not reachable by GitHub. For this local practical, `pollSCM` is the reliable commit-triggered alternative. A public tunnel and suitable Jenkins GitHub/Multibranch plugins are needed for webhook or pull-request triggers.

## Cloud deployment note
This package deploys the container to the local Docker host. Actual AWS/GCP deployment additionally requires a cloud account, target service/cluster, and credentials configured securely in Jenkins. Never put cloud access keys in the Jenkinsfile. The pipeline can be extended for AWS ECS or Google Cloud Run once those details are available.
