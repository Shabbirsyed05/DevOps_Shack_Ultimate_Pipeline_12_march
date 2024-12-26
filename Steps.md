# Phase 1 :

Security Group :
![image](https://github.com/user-attachments/assets/cffc0477-1d90-45d2-a067-809ad248bcb8)
Port	       Description
30,000-32,767 -> Range used for deployment of applications in a Kubernetes cluster
465   Used for sending mail notifications from the Jenkins pipeline to your Gmail address
6443  Required when setting up a Kubernetes cluster
22    Used for accessing virtual machines
443   Used for HTTPS
80    Used for HTTP
3,000-10,000 -> Range used for deploying most applications
25    Used for SMTP server

count : 3 (Master , slave 1 , slave 2) size : t2. medium 25 gb => Ubuntu Server 20.04 LTS

====================================================
# k8s Setup :
#  Setup K8-Cluster using kubeadm [K8 Version-->1.28.1]

### 1. Update System Packages [On Master & Worker Node]

```
sudo apt-get update
```

### 2. Install Docker[On Master & Worker Node]

```
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock
```

### 3. Install Required Dependencies for Kubernetes[On Master & Worker Node]

```
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
```

### 4. Add Kubernetes Repository and GPG Key[On Master & Worker Node]

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 5. Update Package List[On Master & Worker Node]

```
sudo apt update
```

### 6. Install Kubernetes Components[On Master & Worker Node]

```
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
```

### 7. Initialize Kubernetes Master Node [On MasterNode]

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### 8. Configure Kubernetes Cluster [On MasterNode]

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 9. Deploy Networking Solution (Calico) [On MasterNode]

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 10. Deploy Ingress Controller (NGINX) [On MasterNode]

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

=============
after setting up the kubernetes. Scan the kubernetes cluster for any kind of issues. We have many tools like but trivy may not work always.
So we are using kubeaudit

https://github.com/Shopify/kubeaudit/releases

wget link -> master

tar -xvzf kubeaudit_linux_amd64.tar.gz

sudo mv kubeaudit /usr/local/bin/
kubeaudit all

================================================================
# SetUp SonarQube and Nexus :
count : 2 (sonarqube , nexus) size : t2. medium 20 gb

# SonarQube :
# SetUp SonarQube

```
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

```
chmod +x install_docker.sh
```

Then, you can run the script using:

```
./install_docker.sh
```

all users wont be having access
```
sudo chmod 666 /var/run/docker.sock
```

## Create Sonarqube Docker container
To run SonarQube in a Docker container with the provided command, you can follow these steps:

1. Open your terminal or command prompt.

2. Run the following command:

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

This command will download the `sonarqube:lts-community` Docker image from Docker Hub if it's not already available locally. Then, it will create a container named "sonar" from this image, running it in detached mode (`-d` flag) and mapping port 9000 on the host machine to port 9000 in the container (`-p 9000:9000` flag).

3. Access SonarQube by opening a web browser and navigating to `http://VmIP:9000`.

This will start the SonarQube server, and you should be able to access it using the provided URL. If you're running Docker on a remote server or a different port, replace `localhost` with the appropriate hostname or IP address and adjust the port accordingly.

======================================================
# Nexus :
# SetUp Nexus

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

all users wont be having access
```
sudo chmod 666 /var/run/docker.sock
```

## Create Nexus using docker container

To create a Docker container running Nexus 3 and exposing it on port 8081, you can use the following command:

```bash
docker run -d --name nexus -p 8081:8081 sonatype/nexus3:latest
```

This command does the following:

- `-d`: Detaches the container and runs it in the background.
- `--name nexus`: Specifies the name of the container as "nexus".
- `-p 8081:8081`: Maps port 8081 on the host to port 8081 on the container, allowing access to Nexus through port 8081.
- `sonatype/nexus3:latest`: Specifies the Docker image to use for the container, in this case, the latest version of Nexus 3 from the Sonatype repository.

After running this command, Nexus will be accessible on your host machine at http://IP:8081.

## Get Nexus initial password
Your provided commands are correct for accessing the Nexus password stored in the container. Here's a breakdown of the steps:

1. **Get Container ID**: You need to find out the ID of the Nexus container. You can do this by running:

    ```bash
    docker ps
    ```

    This command lists all running containers along with their IDs, among other information.

2. **Access Container's Bash Shell**: Once you have the container ID, you can execute the `docker exec` command to access the container's bash shell:

    ```bash
    docker exec -it <container_ID> /bin/bash
    ```

    Replace `<container_ID>` with the actual ID of the Nexus container.

3. **Navigate to Nexus Directory**: Inside the container's bash shell, navigate to the directory where Nexus stores its configuration:

    ```bash
    cd sonatype-work/nexus3
    ```

4. **View Admin Password**: Finally, you can view the admin password by displaying the contents of the `admin.password` file:

    ```bash
    cat admin.password
    ```

5. **Exit the Container Shell**: Once you have retrieved the password, you can exit the container's bash shell:

    ```bash
    exit
    ```

This process allows you to access the Nexus admin password stored within the container. Make sure to keep this password secure, as it grants administrative access to your Nexus instance.

==================================
# Git :

## Steps to create a private Git repository, generate a personal access token, connect to the repository, and push code to it:

1. **Create a Private Git Repository**:
   - Go to your preferred Git hosting platform (e.g., GitHub, GitLab, Bitbucket).
   - Log in to your account or sign up if you don't have one.
   - Create a new repository and set it as private.

2. **Generate a Personal Access Token**:
   - Navigate to your account settings or profile settings.
   - Look for the "Developer settings" or "Personal access tokens" section.
   - Generate a new token, providing it with the necessary permissions (e.g., repo access).

3. **Clone the Repository Locally**:
   - Open Git Bash or your terminal.
   - Navigate to the directory where you want to clone the repository.
   - Use the `git clone` command followed by the repository's URL. For example:
     ```bash
     git clone <repository_URL>
     ```
   Replace `<repository_URL>` with the URL of your private repository.

4. **Add Your Source Code Files**:
   - Navigate into the cloned repository directory.
   - Paste your source code files or create new ones inside this directory.

5. **Stage and Commit Changes**:
   - Use the `git add` command to stage the changes:
     ```bash
     git add .
     ```
   - Use the `git commit` command to commit the staged changes along with a meaningful message:
     ```bash
     git commit -m "Your commit message here"
     ```

6. **Push Changes to the Repository**:
   - Use the `git push` command to push your committed changes to the remote repository:
     ```bash
     git push
     ```
   - If it's your first time pushing to this repository, you might need to specify the remote and branch:
     ```bash
     git push -u origin master
     ```
   Replace `master` with the branch name if you're pushing to a different branch.

7. **Enter Personal Access Token as Authentication**:
   - When prompted for credentials during the push, enter your username (usually your email) and use your personal access token as the password.

By following these steps, you'll be able to create a private Git repository, connect to it using Git Bash, and push your code changes securely using a personal access token for authentication.

==================================
# Jenkins :
count : 1 (jenkins) size : t2.large size 30 gb

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

==================================

# Pipeline 

```
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    enviornment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/jaiswaladi246/Boardgame.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        
        stage('Publish To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t adijaiswal/boardshack:latest ."
                    }
               }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html adijaiswal/boardshack:latest "
            }
        }
        
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push adijaiswal/boardshack:latest"
                    }
               }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
                        sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                }
            }
        }
        
        
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'shabbirsyed085@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

}
```

==================================

# RBAC :
on master server:

vi svc.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```
```
kubectl create ns webapps
kubectl apply -f svc.yaml
```
vi role.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - secrets
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```
```
kubectl apply -f role.yaml
vi bind.yaml
```
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: webapps 
  kind: ServiceAccount
  name: jenkins 
```
```
kubectl apply -f bind.yaml
vi secret.yaml
```
```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```
```
kubectl apply -f secret.yaml -n webapps

kubectl describe secret mysecretname -n webapps (copy token and paste in jenkins for authentication from jenkins to kubernetes(secret text)(k8-cred))
```
```
cd ~/.kube
ls
cat config (copy the server and update in 'Deploy To Kubernetes' stage => serverUrl: 'https://172.31.41.133:6443)
```


on jenkins server : 

vi k.sh
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

```
sudo chmod +x k.sh
./k.sh
```
```
google account -> app password -> app name (jenkins) -> generate password
jenkins GUI -> manage jenkins -> system  ->  SMTP server (smtp.gmail.com) , port (465) , use ssl(tick) , 
add -> jenkins , username (shabbirsyed0786@gmail.com , password (generated one))

jenkins GUI -> manage jenkins -> system  -> E-mail Notification -> SMTP server (smtp.gmail.com) , uss ssl (tick) , port (465),
Use SMTP Authentication -> shabbirsyed0786@gmail.com (name), password generated one , Test configuration by sending test e-mail

```

## before doing build . in nexus -> maven-releases -> delete the folder and run the build or else it will be failing.
```
copy the slave-ip_address:port (u can get the port from the console output in the ending)

bugs , bunny (username and password)
for normal user -> u can addd but cant edit and delete

manager role:
daffy , duck
```
===============================================
# Monitor server :
````
monitor ec2:
size : t2.large , size: 20

sudo apt update
(https://prometheus.io/download/)
copy amd64 -> wget link
tar -xvf filename
rm -rf tar
cd promethus_file
./prometheus &

copy monitor_ip_address:9090

install grafana (search)
https://grafana.com/grafana/download

copy monitor_ip_address:3000 (for grafana) admin,admin

search blackbox in (https://prometheus.io/download/)
wget link
tar -xvf blackbox_filename
ls
cd blackbox_file
ls
./blackbox_ &

copy monitor_ip_address:9115

cd ..
cd prometheus
ls
vi prometheus.yml

https://github.com/prometheus/blackbox_exporter

vi prometheus.yml (need to add below code -> below static_config -> targets, change ip-addresss in replacement , change "13.201.94.63:30528 " with previously opened application ip i.e slave_ip_address:port_in_jenkins_console)
```
```
- job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - https://prometheus.io   # Target to probe with https.
        - http://13.201.94.63:30528 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 3.108.67.147:9115  # The blackbox exporter's real hostname:port.

```

```
pgrep prometheus
kill pid_number
./prometheus &

prometheus webconsole -> status -> targets
http://3.108.67.147:9090/targets (u can see they will come up)

Grafana GUI -> Connections ->  Data sources -> Add data source -> prometheus ->  add promethues url  -> save & test
Grafana GUI -> + icon -> import dashboard ->
blackbox grafana dashboard -> copy ip to clipboard  , datasource as promethueus

jenkins -> plugins -> prometheus metrics and restart jenkins.
```
jenkins server
```
search -> node_exporter from https://prometheus.io/download/
wget link
tar -xvf node
cd node_
./node_explorer &

copy jenkins_ip_address:9100

Jenkins GUI
Dashboard -> Manage Jenkins ->System -> find prometheus (if u want any changes)

```
vi prometheus.yml (below targets:[localhost:9000])
```
- job_name: 'node_exporter'
  static_configs:
  - targets: ['jenkins_ip_address:9100']
- job_name: 'jenkins'
  metrics_path: '/prometheus'
  static_configs:
  - targets: ['jenkins_ip_address:Jenkins_port']
```
pgrep prometheus
kill pid
./prometheus &

refresh prometheus webpage

node exporter grafana -> 1860

grafana -> dashboard -> new -> import 
 
