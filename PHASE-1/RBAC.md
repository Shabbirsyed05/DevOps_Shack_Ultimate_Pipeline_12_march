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

google account -> app password -> app name (jenkins) -> generate password
jenkins GUI -> manage jenkins -> system  ->  SMTP server (smtp.gmail.com) , port (465) , use ssl(tick) , 
add -> jenkins , username (shabbirsyed0786@gmail.com , password (generated one))

jenkins GUI -> manage jenkins -> system  -> E-mail Notification -> SMTP server (smtp.gmail.com) , uss ssl (tick) , port (465),
Use SMTP Authentication -> shabbirsyed0786@gmail.com (name), password generated one , Test configuration by sending test e-mail



