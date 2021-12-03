# All command in workshop

CheatSheet for Final Exam INT492 Devops

## Prepare for exam

```bash
# Set default to namespace
kubectl config set-context $(kubectl config current-context) --namespace=student[X]
kubectl get deployments

## if can not see deployment, must see in exam doc for solution

# get pod and enter bash in pod
kubectl get pod
kubectl exec -it student[X]-result-[HASH] -- bash

# cd to exam directory and create directory 1-20
cd /exam
mkdir {1..20}
ls -la

# for copy file from cloud shell to pod
kubectl cp [File/Directory] [Pod name]:[Path]
## copy for file
kubectl cp output.txt student[X]-result-[HASH]:/exam/3/output.txt
## copy for directory
kubectl cp 1/ student[X]-result-[HASH]:/exam/
```

## Suggest for see tree structure & install docker-compose

```bash
# install tree
sudo apt install tree

# install docker-compose
sudo python3 -m pip install --upgrade pip
sudo CRYPTOGRAPHY_DONT_BUILD_RUST=1 python3 -m pip install --upgrade docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
exec bash
docker-compose version
```

## Git & GitHub

```bash
# git clone
## folder name optional with no folder name it will create folder name by repo name
git clone [URL] [Folder name]

# git branch
## create branch
git branch [Branch name]
## check branch
git branch
## checkout to branch
git checkout [Branch name]

# git commit
## check file that changed
git status
## add to commit
git add .
git add [File path]
## commit with message
git commit -m "[message]"
```

* You can see addition detail in Workshop/01-git.md <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/01-git.md>

## Linux

```bash
# create directory
## you can make multiple directory in one line
mkdir [Directory name], ...

# copy file
## you can add -i for ask confirmation or -r for copy whole directory
cp <source file> <target file>
cp -r <source dir> <target dir>
## you can cp with multiple file to target directory
cp file1 file2 ... <directory>

# move file
## change name to new file 
mv <old name> <new name>

# remove file and directory
## file
rm <file name>
rm -i <file name>

# scan pattern
grep <pattern> <file>

# change permission
## change with number
### read = 4, write = 2, execute = 1, no permission = 0
### first digit to owner, second digit to group, last digit to other
sudo chmod 777 <file name>

## change owner and group
sudo chgrp <group name> <file name>
sudo chown <owner name> <file name>

## change with name
### g is group permission and add execute to group permisssion
### u is owner, g is group, o is other and a is all user
### + means add, - mean minus and = means only permission that you want
sudo chmod g+x <file name>
sudo chmod g-x <file name>
sudo chmod g=r <file name> # group can read only

# how to write continue in file
cal 2020 > output.txt # it will create output.txt
cal 2021 >> output.txt # it will write continue from old file
```

* Find more in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/02-linux.md>

## Docker

You can search for docker image in Docker Hub <https://hub.docker.com>
You can see Dockerfile structure in Docker Hub and use it exam.

```bash
# see docker container
## running container
docker ps
## running and stopped container
docker ps -a

# build image from docker file
## you must in directory that have Dockerfile.yaml
docker build -t <image name> .

# see docker image
docker images

# delete docker image
docker rm -f <image name>

# log docker container
docker logs -f <container name>

# run docker container with port and environment
docker run -d --name <container name> -p [External port]:[Internal port] -e [Key]:[Value] [Image name]
```

* Find more in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/03-docker.md>

## Docker compose

* In google cloud platform, you must install docker compose

```bash
sudo python3 -m pip install --upgrade pip
sudo CRYPTOGRAPHY_DONT_BUILD_RUST=1 python3 -m pip install --upgrade docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
exec bash
docker-compose version
```

* You must create `docker-compose.yml` or `docker-compose.yaml` in that directory

### Structure
!!! be careful with tab indent, it will affect in your build process !!!

```bash
# docker-compose in beginning
services:
  ratings:
    build: .
    image: ghcr.io/[GITHUB_USER]/[REPO NAME]:[BRANCH or TAG]
    ports:
      - "8080:8080"
    environment:
      SERVICE_VERSION: v1

# docker-compose in rating
services:
  ratings:
    build: .
    image: ghcr.io/supakit25433/bookinfo-ratings:dev
    ports:
      - "8080:8080"
    environment:
      SERVICE_VERSION: v2
      MONGO_DB_URL: mongodb://mongodb:27017/ratings
      MONGO_DB_USERNAME: ratings
      MONGO_DB_PASSWORD: CHANGEME
  mongodb:
    image: bitnami/mongodb:5.0.2-debian-10-r2
    volumes:
      - "./databases:/docker-entrypoint-initdb.d"
    environment:
      MONGODB_ROOT_PASSWORD: CHANGEME
      MONGODB_USERNAME: ratings
      MONGODB_PASSWORD: CHANGEME
      MONGODB_DATABASE: ratings
```

You can run by this command. But you must delete running container that have same name

```bash
docker-compose up
```

* Utility Command
```bash
docker-compose build # build only
docker-compose up --build # build and run
docker-compose -f [custom yaml] up # run with custom yaml
docker-compose up -d # run in background
docker-compose ps # show status container
docker-compose stop # stop all
docker-compose start # start all
docker-compose restart # restart
docker-compose logs # log all container
docker-compose logs -f [NAME] # log with specific container
docker-compose top # show all process
docker-compose images # show image that build in docker-compose
docker-compose stop # stop and clean
```

* Code before this line is will change or add some file. You can see details in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/04-docker-compose.md>

# Kubernetes (k8s)

You must get kubeconfig but it will depands on Exam time. Example below

```bash
# get kubeconfig
gcloud container clusters get-credentials k8s --project int492-sit-kmutt --zone asia-southeast1-a
# check if it connect to cluster
kubectl version
```

* Set Namespace

```bash
# Show all namespace in cluster
kubectl get namespaces

# Show current cluster config
kubectl config get-contexts

# Create namespace
kubectl create namespace [NAMESPACE]

# Set default namespace
kubectl config set-context $(kubectl config current-context) --namespace=[NAMESPACE]
```

* Create pod and deployment manual

```bash
# After this command, it will create deployment and pod
kubectl create deployment [NAME] --image=[IMAGE]

# You can see status of pod, deployment
kubectl get pod,deployment[, ...]

# Describe detail
kubectl describe [TYPE] [NAME]
kubectl describe pod nginx
```

You can delete pod but deployment will create new pod for that service

* Create service manual

```bash
kubectl expose deployment nginx --type ClusterIP --port 80 --name nginx-cip

# (1) set with proxy
kubectl proxy --port=8080
# (2) set port forward directly
kubectl port-forward service/nginx-cip 8080:80
```

* Scale service and change docker image

```bash
# change scale with deployment
kubectl scale deployment [NAME] --replicas=[NUMBER]

# change docker image
kubectl set image deployment [NAME] [OLD IMAGE]=[NEW IMAGE]

# see change with port-forward command
watch -n1 kubectl get [TYPE]
```

* Rollback deployment
```bash
# See history
kubectl rollout history deployment [NAME]
kubectl rollout undo deployment [NAME]
```

* Label and Selector
```bash
# create for test label and selector
kubectl create deployment apache --image=httpd:2.4-alpine
kubectl scale deployment apache --replicas=3

# see label and selector
kubectl describe [TYPE] [NAME]

# set selector
kubectl set selector service nginx-cip 'app=apache'
kubectl set selector service nginx-cip 'app=nginx'
```

* k8s Utilities Command

```bash
# log
kubectl logs -f [NAME or ID]

# get inside container
kubectl exec -it [NAME or ID] -- sh
```

* How to clear everything
```bash
kubectl delete [TYPE] [NAME or ID], ...
kubectl delete namespace [NAMESPACE]
```

* see more details in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/05-k8s-cli.md>

# k8s manifest file

You must prepare k8s environment everytime
- get cluster
- create namespace
- set default namespace

```bash
# In exam, it will depands on teacher
gcloud container clusters get-credentials k8s --project int492-sit-kmutt --zone asia-southeast1-a
kubectl create namespace [NAMESPACE]
kubectl config set-context $(kubectl config current-context) --namespace=[NAMESPACE]
```

* Create and Initial

You must make directory in your folder `k8s` in this directory will contain pod, deployment, service

* Example pod, deployment, service

```bash
# pod
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: student[X]-manifest
spec:
  containers:
  - name: busybox
    image: busybox
    command:
    - sleep
    - "3600"

# deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache
  namespace: student[X]-manifest
  labels:
    app: apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - image: httpd:2.4.43-alpine
        name: apache

# service
apiVersion: v1
kind: Service
metadata:
  name: apache
  namespace: student[X]-manifest
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: apache

```

You can use manifest file by this command

```bash
cd k8s/
kubectl apply -f [FILE], -f [FILE]
kubectl port-forward service/apache 8080:80
```

## secret manifest file use cases

see more: <https://kubernetes.io/docs/concepts/configuration/secret/>

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

* Clean
```bash
# You must in directory that have manifest file
kubectl delete -f .
kubectl delete namespace [NAMESPACE]
```

* See more details in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/06-k8s-manifest.md>

# Deploy ratings on k8s

You must prepare k8s environment everytime
- get cluster
- create namespace
- set default namespace

```bash
# In exam, it will depands on teacher
gcloud container clusters get-credentials k8s --project int492-sit-kmutt --zone asia-southeast1-a
kubectl create namespace [NAMESPACE]
kubectl config set-context $(kubectl config current-context) --namespace=[NAMESPACE]
```

You must install docker-compose

```bash
sudo python3 -m pip install --upgrade pip
sudo CRYPTOGRAPHY_DONT_BUILD_RUST=1 python3 -m pip install --upgrade docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
exec bash
docker-compose version
```

* Prepare GitHub Container Registry

Go to Github/Settings/Tokens or Go to GitHub/Settings/Developer settings/Personal access tokens > Generate new token

- set expire
- select scope > repo & write:packages
- copy token

* Build and push docker image
```bash
cd [Directory]
docker-compose build

export TOKEN=[TOKEN from GitHub]
export GITHUB_USER=[Username]

echo $TOKEN | docker login ghcr.io --password-stdin --username $GITHUB_USER
docker push ghcr.io/[GitHub User]/[REPO NAME]:[TAG must same in docker-compose.yaml]
```

* Create Secret to pull docker image
!!!! It will depends on teacher. !!!!
```bash
kubectl create secret generic [NAME] \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson
```

* Example deployment, service, ingress
```bash
# deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookinfo-dev-ratings
  namespace: student[X]-bookinfo-dev
  labels:
    app: bookinfo-dev-ratings
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookinfo-dev-ratings
  template:
    metadata:
      labels:
        app: bookinfo-dev-ratings
    spec:
      containers:
      - name: bookinfo-dev-ratings
        image: ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
        imagePullPolicy: Always
        env:
        - name: SERVICE_VERSION
          value: v1
      imagePullSecrets:
      - name: registry-bookinfo

# service
apiVersion: v1
kind: Service
metadata:
  name: bookinfo-dev-ratings
  namespace: student[X]-bookinfo-dev
spec:
  type: ClusterIP
  ports:
  - port: 8080
  selector:
    app: bookinfo-dev-ratings

# ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: bookinfo-dev-ratings
  namespace: student[X]-bookinfo-dev
spec:
  rules:
  - host: sitkmutt.bookinfo.dev.opsta.net
    http:
      paths:
      - path: /student[X]/ratings(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: bookinfo-dev-ratings
            port:
              number: 8080
```

and you can create deployment by
```bash
# you must in directory
kubectl apply -f k8s/
```

* You can see details in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/07-k8s-rating.md>

# Deploy MongoDB with Helm Chart

* You can see chart in <https://hub.kubeapps.com/charts>
* You can see template for chart value in <https://github.com/helm/charts/tree/master/stable>

## Use bitnami chart

```bash
# Add and update
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install
helm install mongodb bitnami/mongodb --set persistence.enabled=false

# Connect
export MONGODB_ROOT_PASSWORD=$(kubectl get secret mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
kubectl run mongodb-client --rm --tty -i --restart='Never' --image bitnami/mongodb:5.0.3-debian-10-r30   --command -- mongo admin --host mongodb --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD
```

## Use custom config MongoDB with Helm Value

```bash
# create secret for mongodb
kubectl create secret generic bookinfo-dev-ratings-mongodb-secret --from-literal=mongodb-password=CHANGEME --from-literal=mongodb-root-password=CHANGEME
```

make directory `k8s/helm-values` and create `values-bookinfo-dev-ratings-mongodb.yaml` file in `helm-values`

```bash
image:
  tag: 5.0.3-debian-10-r30
auth:
  enabled: true
  username: ratings
  database: ratings
  existingSecret: bookinfo-dev-ratings-mongodb-secret
persistence:
  enabled: false
initdbScriptsConfigMap: bookinfo-dev-ratings-mongodb-initdb
```

```bash
# create configmap (something will change in exam)
ubectl create configmap bookinfo-dev-ratings-mongodb-initdb --from-file=databases/ratings_data.json --from-file=databases/script.sh

# deploy mongodb
helm install -f k8s/helm-values/values-bookinfo-dev-ratings-mongodb.yaml  bookinfo-dev-ratings-mongodb bitnami/mongodb --version 10.28.4
```

* update rating service manifest file

```bash
...
        env:
        - name: SERVICE_VERSION
          value: v2
        - name: MONGO_DB_URL
          value: mongodb://bookinfo-dev-ratings-mongodb:27017/ratings
        - name: MONGO_DB_USERNAME
          value: ratings
        - name: MONGO_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: bookinfo-dev-ratings-mongodb-secret
              key: mongodb-password
...
```

and run `kubectl apply -f k8s/`

* See more detail in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/08-helm-mongodb.md>

# Deploy ratings service with helm chart

* Preparation
- Create namespace
- Set default namespace
- Create k8s imagePullSecrets
- Create k8s Secret for mongodb password
- Create k8s ConfigMap
- Deploy MongoDB with helm

* Create Helm Chart

make directory `ratings/k8s/helm` and create file `Chart.yaml` in helm directory

```bash
apiVersion: v1
description: Bookinfo Ratings Service Helm Chart
name: bookinfo-ratings
version: 1.0.0
appVersion: 1.0.0
home: http://sitkmutt.bookinfo.dev.opsta.net/student[X]/ratings
maintainers:
  - name: student[X]
    email: [XXX]@mail.kmutt.ac.th
sources:
  - https://github.com/[GITHUB_USER]/sitkmutt-bookinfo-ratings
```

make directory `ratings/k8s/helm/templates` and deployment, service, ingress from `k8s/` into `k8s/helm/templates`

test deploy ratings service
```bash
helm install bookinfo-dev-ratings k8s/helm
```

* Create Helm Value file for ratings service

make `k8s/helm-values` and create `values-bookinfo-dev-ratings.yaml`

```bash
ratings:
  namespace: student[X]-bookinfo-dev
  replicas: 1
  imagePullSecrets: registry-bookinfo
  port: 8080
  image: ghcr.io/[GITHUB_USER]/bookinfo-ratings
  tag: dev
ingress:
  host: sitkmutt.bookinfo.dev.opsta.net
  path: "/student[X]/ratings(/|$)(.*)"
  serviceType: ClusterIP
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
extraEnv:
  SERVICE_VERSION: v2
  MONGO_DB_URL: mongodb://bookinfo-dev-ratings-mongodb:27017/ratings
  MONGO_DB_USERNAME: ratings
extraEnvSecret:
  MONGO_DB_PASSWORD:
    bookinfo-dev-ratings-mongodb-secret: mongodb-password
```

replace in deployment, service and ingress with `{{.Release.Name}}, {{.Values.ratings.*}}, {{.Values.ingress.*}}`

and you can use if or loop to replace value in file

```bash
# example 1
  {{- if .Values.ingress.annotations }}
  annotations:
    {{- range $key, $value := .Values.ingress.annotations }}
    {{ $key }}: {{ $value | quote }}
    {{- end }}
  {{- end }}

# example 2
        {{- if or (.Values.extraEnv) (.Values.extraEnvSecret) }}
        env:

        {{- if .Values.extraEnv }}
        {{- range $key, $value := .Values.extraEnv }}
        - name: {{ $key }}
          value: {{ $value | quote }}
        {{- end }}
        {{- end }}

        {{- if .Values.extraEnvSecret }}
        {{- range $key, $value := .Values.extraEnvSecret }}
        {{- range $secretName, $secretValue := $value }}
        - name: {{ $key }}
          valueFrom:
            secretKeyRef:
              name: {{ $secretName }}
              key: {{ $secretValue | quote }}
        {{- end }}
        {{- end }}
        {{- end }}

        {{- end }}
```

after replace you can run it with

```bash
# first init
helm install -f k8s/helm-values/values-bookinfo-dev-ratings.yaml bookinfo-dev-ratings k8s/helm

# after init
helm upgrade -f k8s/helm-values/values-bookinfo-dev-ratings.yaml bookinfo-dev-ratings k8s/helm
```

* You can see details in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/09-helm-rating.md>