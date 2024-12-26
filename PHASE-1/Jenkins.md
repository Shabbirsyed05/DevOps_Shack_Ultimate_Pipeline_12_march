## Installing Jenkins on Ubuntu


```bash
#!/bin/bash

# Install OpenJDK 17 JRE Headless
sudo apt install openjdk-17-jre-headless -y

# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y
```

Save this script in a file, for example, `install_jenkins.sh`, and make it executable using:

```bash
chmod +x install_jenkins.sh
```

Then, you can run the script using:

```bash
./install_jenkins.sh
```

This script will automate the installation process of OpenJDK 17 JRE Headless and Jenkins.


## Install docker for future use

```bash
#!/bin/bash

# Update package manager repositories
sudo apt-get update

# Install necessary dependencies
sudo apt-get install -y ca-certificates curl

# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package manager repositories
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
```

Save this script in a file, for example, `install_docker.sh`, and make it executable using:

```bash
chmod +x install_docker.sh
```

Then, you can run the script using:

```bash
./install_docker.sh
```
for getting access for other users
```
sudo chmod 666 /var/run/docker.sock
```
#### Plugins :
```
Jenkins Plugins :
jdk -> Eclipse Termurin installer 
maven -> config file provider , pipeline maven Integration , maven integration
Sonarqube scanner
docker -> docker , docker pipeline 
kubernetes -> kubernetes , kubernetes CLI , kubernetes credentials , kubernetes client API

Manage jenkins -> Tools =>
jdk -> jdk11 (name)-> install from adoptium ,jdk 17.09 
sonarqube-scanner -> sonar-scanner (name)
maven -> maven3 (name) -> version : 3.61
docker -> docker (name) -> docker.com (latest)
prometheus metrics
```

```
Jenkins GUI -> Manage Jenkins -> Tools =>
JDK name (jdk17) , Install automatically (add from adoptium ) -> jdk-17.0.9+9
SonarQube Scanner -> name (sonar-scanner), install automatically (lastest)
maven -> maven3(name), version (3.6.1)
docker -> docker (name), install automatically (download from docker.com)
```
for installing trivy
```
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```
```
trivy --version
sonarqube =>  adminsitrator -> security -> users -> generate token
```
```
Install trivy
generate toke for git and sonarqube
add git cred and sonar credentials in jenkins credentials
manage jenkins -> system =>
sonarqube server -> sonar(name) , server url -> https://ip_address_public:9000, auth -> sonar-token from drop-down
```
```
sonarqube => Administrator -> configuration -> webhooks -> create ->
name :jenkins , url : http://jenkins_ip:8080/sonarqube-webhook/
```
```
Nexus => maven-releases and maven-snapshots  -> copy and paste in pom.xml file
```
```
Manage Jenkins -> Managed files -> add a new config -> Global Maven settings.xml -> global-settings (ID ) -> update the content in with maven-release , maven-snapshot with username and password.
```
```
add docker credentials in jenkins credentials
```
```
Jenkins Plugins :
jdk -> Eclipse Termurin installer 
maven -> config file provider , pipeline maven Integration , maven integration
Sonarqube scanner
docker -> docker , docker pipeline 
kubernetes -> kubernetes , kubernetes CLI , kubernetes credentials , kubernetes client API

Manage jenkins -> Tools =>
jdk -> jdk11 (name)-> install from adoptium ,jdk 17.09 
sonarqube-scanner -> sonar-scanner (name)
maven -> maven3 (name) -> version : 3.61
docker -> docker (name) -> docker.com (latest)
prometheus metrics
```
