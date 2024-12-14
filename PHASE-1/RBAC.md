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
Monitor server :
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
 
