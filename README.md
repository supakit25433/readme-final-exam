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

* About set protect from direct merge request

You can see addition detail in Workshop/01-git.md <https://github.com/opsta-education/int492-2021-devops-workshop/blob/main/docs/01-git.md>

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

* File structure (depand on instruction of that image)

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