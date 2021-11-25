# All command in workshop

CheatSheet for Final Exam INT492 Devops

## Git & GitHub

* Maybe not use in Final Exam

* Set git config

```bash
git config --global user.name “[Your name]”
git config --global user.email “[Your email]”
// check config
git config --list
```

* Init git to folder

```bash
cd [FOLDER]
git init
```

* Add first commit

```bash
# You can use one of below
git add .
git add [FILE]

# Check status
git status

# First commit
git commit -m "[MESSAGE]"
```

* Push repo to GitHub

```bash
# generate SSH
ssh-keygen
# Copy public key
cat ~/.ssh/id_rsa.pub
```

After you copy ssh key, you will go to `GitHub Setting / SSH and GPG Key`. Copy last part in SSH to `Title` and first part for `Key`.

After you add SSH Key success, you must create new repository on GitHub.

```bash
# Add SSH of repo to folder
git remote add origin [YOUR SSH IN REPO]
# To see remote repository has been added
git remote -v
# Set default branch to main
git branch -M main
# Push code to GitHub
git push -u origin main
```

* How to clone project

```bash
git clone git@github.com:[USERNAME]/[REPO NAME].git [FOLDER NAME]
```

* How to fork branch

```bash
# create branch
git branch [BRANCH NAME]
# checkout to branch
git checkout [BRANCH NAME]
# push to GitHub
git push --set-upstream origin [BRANCH NAME]
```

* How to see diff between current and old commit

```bash
git diff
```

* You can see addition detail in Workshop/01-git.md <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/01-git.md>

## Linux

```bash
# Show all file and permission in that directory
ls -la

# Show history
history

# Show what that command can do
whatis [COMMAND]
man [COMMAND]
[COMMAND] --help

# Show content
cat [FILE]

# use spacebar to next
more [FILE]

# use arrow or page up/down
less [FILE]

# default 10 lines
head [FILE]

# 5 lines
head -n 5 [FILE] 

# 10 lines
tail [FILE] 

# open another tab
tail -f [FILE]

# Search in File
grep [PATTERN] [FILE]
```

* Find more in <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/02-linux.md>

## Docker

```bash
# see image on machine
docker images

# pull image lastest version from Docker Hub
docker pull [IMAGE]
# pull with version
docker pull [IMAGE]:[VERSION]


# You can run with command for that image
docker run [IMAGE] [COMMAND]
# It will run bash from that image
docker run -i -t ubuntu bash

# Show only running container
docker ps
# Show running and stopped container
docker ps -a
# Run container with specific name
docker run --name [Container Name] [Image] [Command]

# Delete container
## by name
docker rm [Container Name]
## by ID
docker rm [part of ID]

# Run Docker as deamon and expose port
## run in front show ui
docker run [IMAGE]:[VERSION]
## run in background
docker run -d [IMAGE]:[VERSION]
## run in background and expose port
docker run -d -p 8080:80 [IMAGE]:[VERSION]

### You cannot use port that already exposed but can use other port that will create new container

# Utilities command
## Rename container
docker rename [NEW NAME] [OLD NAME]
## Go inside running container
docker exec -it [NAME] [COMMAND] # docker exec -it nginx sh
## Show container process
docker top [NAME]
## Show log
docker logs [NAME]
## Show log real-time
docker logs -f [NAME]
## Show resource
docker stats
## Show container all metadata
docker inspect [NAME]

# Delete All Container
docker rm -f ${docker ps -aq}
```

### File structure (depand on instruction of that image)

```bash
FROM node:16.8.0-alpine3.12

WORKDIR /usr/src/app/

COPY src/ /usr/src/app/
RUN npm install

EXPOSE 8080

CMD ["node", "/usr/src/app/ratings.js", "8080"]
```

1. Start from build images
```bash
# must in directory that want to use
docker build -t [NAME] [FILE] ## docker build -t ratings .
```
2. Check image
```bash
docker images
```
3. Run container from image with expose port
```bash
docker run -d --name [NAME] -p [External Port]:[Internal Port] [Image name]
```
4. Run container from image with expose port and environment variable
```bash
docker run -d --name [NAME] -p [External Port]:[Internal Port] -e [Name]:[Value] (...) [Image name]
``` 

* How to run MongoDB Container
```bash
# no initial database
docker run -d --name mongodb -p 27017:27017 bitnami/mongodb:5.0.2-debian-10-r2
# with initial database
docker run -d --name mongodb -p 27017:27017 -v $(pwd)/databases:/docker-entrypoint-initdb.d bitnami/mongodb:5.0.2-debian-10-r2

# How to use in rating
docker run -d --name ratings -p 8080:8080 --link mongodb:mongodb -e SERVICE_VERSION=v2 -e 'MONGO_DB_URL=mongodb://mongodb:27017/ratings' ratings
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
!!! be careful with tab indent, it will affect in your build process!!!

```bash
services:
  ratings:
    build: .
    image: ghcr.io/[GITHUB_USER]/[REPO NAME]:[BRANCH or TAG]
    ports:
      - "8080:8080"
    environment:
      SERVICE_VERSION: v1
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

* Example docker-compose.yaml for ratings
```bash
services:
  ratings:
    build: .
    image: ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
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

* Clean
```bash
# You must in directory that have manifest file
kubectl delete -f .
kubectl delete namespace [NAMESPACE]
```

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