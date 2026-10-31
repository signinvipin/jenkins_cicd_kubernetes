# jenkins_cicd_kubernetes

A Jenkins declarative pipeline that builds a Spring Boot (Java 17) app, runs SonarQube analysis, builds and pushes a Docker image, and deploys it to Kubernetes.

## Pipeline stages (`jenkins_declarative_script.groovy`)
1. Clone the Git repository
2. Build and SonarQube analysis (scanner token from Jenkins credentials)
3. Build the Java app with Maven
4. Docker build and push to Docker Hub (Docker Pipeline plugin)
5. Deploy to Kubernetes: `envsubst` fills the image tag into `app-deployment.yaml`, then the Kubernetes deploy step applies it

### Target workflow
```text
Developer pushes to GitHub (main)
            ↓
     GitHub webhook triggers Jenkins
            ↓
  [Clone Git] → [Run Tests] → [Build Docker Image]
                                ↓
                         [Push to Registry]
                                ↓
                  [Deploy to K8s via YAML]
                                ↓
         [Staging] → [Approval] → [Production]
```
The pipeline in this repository covers clone, build, image push and deploy. Staging and approval stages are not implemented here.

## Files
| File | Purpose |
|---|---|
| `jenkins_declarative_script.groovy` | The Jenkins pipeline |
| `Dockerfile` | Alpine + OpenJDK 17 image that runs the built JAR |
| `app-deployment.yaml` | Kubernetes Deployment and LoadBalancer Service |
| `nexus.yaml` | Nexus 3 deployment on Kubernetes |
| `docker-compose.yaml` | SonarQube with a PostgreSQL database |
| `jenkins_cicd_setup.txt` | Setup notes: Java 17, Maven, Jenkins, Docker, SonarQube scanner, Git |
| `kubernetes.txt` | Troubleshooting commands (pod logs, image platform checks, Jenkins docker group) |
| `pom.xml`, `src/` | Spring Boot 3.1.4 sample app |
| `Custom_Plugin_k8s_Jenkins/` | Optional: `kubernetes-cd` plugin file and its source note |

## Requirements
- Jenkins with the SonarQube Scanner, Docker Pipeline and Kubernetes CD (`kubernetes-cd`) plugins. `Custom_Plugin_k8s_Jenkins/` is optional: it holds a copy of the `kubernetes-cd` plugin (`.hpi`) and a note on where it came from. You can also install the plugin from the Jenkins plugin manager.
- Jenkins credentials: `git-credentials`, `sonar-credentials`, `dockerhub-credentials`, and a kubeconfig named `jenkinsCluster`
- A SonarQube server (see `docker-compose.yaml`) and a Kubernetes cluster

## Run SonarQube locally
```bash
cp .env.example .env      # then set DB_PASSWORD in .env
docker compose up -d
```

## Notes
- The SonarQube address in the pipeline is masked (`3.134.xx.xx`). Replace it with your own SonarQube host.
- In the Jenkins job, set the Script Path to `jenkins_declarative_script.groovy`.
- The pipeline checks out this repository (`jenkins-cicd-kubernetes`) from the `main` branch.
