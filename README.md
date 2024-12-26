phase 1 :
```
NWE -> private , isolated , secure
k8s -> Deploy appli (scan it)
vm -> sonar, Nexus, Jenkins, Monitoring
```
Phase 2 :
```
Git Repo (private)
Push our source code
visible in repo (from above one)
```
Phase 3 :
```
cicd pipeline (Best Practices and security measures)
Deployment
Mail Notificatins
```
Phase 4 :
```
Monitoring appli -> system level (cpu, ram)
		     website level (traffic)
```
```
- Compilation  ->	Compiling the source code to check for syntax errors
Unit Testing ->	Running test cases to check the functionality of the code
Code Quality Check -> Using SonarCube to check for bugs, issues, and code smell
Vulnerability Scan ->	Scanning the source code for sensitive data and vulnerabilities
Building and Packaging -> Building and packaging the application to get the application artifact
Publishing to Nexus -> Publishing the application artifact to the Nexus repository
Creating a Docker Image -> Creating a Docker image and tagging it properly
Scanning the Docker Image -> Using Trivy to scan the Docker image for vulnerabilities
Pushing to Docker Hub -> Pushing the Docker image to Docker Hub
Deploying to Kubernetes	-> Deploying the application to the Kubernetes cluster
Monitoring -> Monitoring the application using Blackbox exporter and Grafana
```
