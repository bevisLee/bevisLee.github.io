---
layout: post
title: "Qwiklabs - Introduction to Docker"
comments: true
categories : [DE-Cloud]
tags: [Docker]

---


## 들어가기

* Docker는 응용 프로그램을 개발, 운송 및 실행하기 위한 개방형 플랫폼입니다. Docker를 사용하면 인프라에서 응용 프로그램을 분리하고, 관리되는 응용 프로그램처럼 인프라를 처리 할 수 있습니다. 
* Docker는 커널 컨테이너 기능을 워크 플로우와 결합하여 응용 프로그램을 관리하고 배포하는데 도움을 줍니다.
* Docker 컨테이너는 Kubernetes에서 직접 사용할 수 있으므로, Kubernetes 엔진에서 쉽게 실행 할 수 있습니다. Docker의 핵심을 배우면 Kubernetes 및 컨테이너 응용 프로그램 개발을 시작할 수 있습니다.
 
## Intro

* 도커에 사용 경험이 적어, 퀵랩에 나온 내용을 실행하고, 결과를 적어 놓는 방식으로 포스팅하고자 합니다.

#### Docker run hello-world : Docker 이미지 생성
* 이 간단한 컨테이너가 `Hello from Docker!` 화면으로 돌아감
* 명령은 간단하지만 수행된 단계의 수를 출력에 표시
* docker 데몬은 hello-world 이미지를 검색하고, 로컬에 이미지가 없다면 Docker Hub라는 공개 레지스트르에서 이미지를 가져와서 컨테이너 이미지를 생성

```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker run hello-world
```

#### docker images : 가져온 이미지를 정보 출력

```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker images
```


#### docker ps : 실행중인 컨테이너 정보 출력
* 실행중인 컨테이너 출력


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker ps
```


* 실행완료까지 포함된 컨테이너 출력


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker ps -a
```


## Build
* Dockerfile 생성


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ cat > Dockerfile << EOF

# Use an official Node runtime as the parent image
FROM node:6
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Make the container's port 80 available to the outside world
EXPOSE 80
# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```


* app.js 생성


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ cat > app.js << EOF

const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
     res.statusCode = 200;
       res.setHeader('Content-Type', 'text/plain');
         res.end('Hello World\n');
 });

 server.listen(port, hostname, () => {
     console.log('Server running at http://%s:%s/', hostname, port);
 });

 process.on('SIGINT', function() {
     console.log('Caught interrupt signal and will exit');
     process.exit();
 });
EOF
```


* Docker 이미지 생성
	* 실행을 완료하는데 몇분 정도 걸림


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker build -t node-app:0.1 .

Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM node:6
6: Pulling from library/node
cd8eada9c7bb: Pull complete
c2677faec825: Pull complete
fcce419a96b1: Pull complete
045b51e26e75: Pull complete
3b969ad6f147: Pull complete
a81151df0bcd: Pull complete
90106fd3fb47: Pull complete
ae080e816dd2: Pull complete
Digest: sha256:337a12f4c9da661e9f373c49470668bbcae19fa1accd52ffef301041746b365e
Status: Downloaded newer image for node:6
 ---> 13f9ef525ab4
Step 2/5 : WORKDIR /app
Removing intermediate container 5a74fda5341f
 ---> 453407bf6c80
Step 3/5 : ADD . /app
 ---> 9df5344ec7fa
Step 4/5 : EXPOSE 80
 ---> Running in 917aa3b7095c
Removing intermediate container 917aa3b7095c
 ---> cf751b50e4c2
Step 5/5 : CMD ["node", "app.js"]
 ---> Running in 249dac45d748
Removing intermediate container 249dac45d748
 ---> 97b36d414e1c
Successfully built 97b36d414e1c
Successfully tagged node-app:0.1
```


* 빌드한 Docker 이미지 정보 출력


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker images
```


## Run
* 컨테이너 실행

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker run -p 4000:80 --name my-app node-app:0.1
 
Server running at http://0.0.0.0:80/
```


* 다른 터미널을 열어, 서버 테스트

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ curl http://localhost:4000
 
Hello world
```


* 컨테이너 종료/삭제

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker stop my-app && docker rm my-app
 
my-app
my-app
```


* 컨테이너 백그라운드 실행, 컨테이너 출력

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker run -p 4000:80 --name my-app -d node-app:0.1

> docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
d26e0a8a67e0        node-app:0.1        "node app.js"       7 seconds ago       Up 6 seconds        0.0.0.0:4000->80/tcp   my-app
```


* 컨테이너 log 출력
	* 컨테이너 id가 `d26e0a8a67e0` 인 경우 `docker logs d26`로 실행 가능
	* 컨테이너가 실행중일 때 로그를 출력하려면 -f

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker logs [container_id]

> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker logs -f [container_id]
```


* 기존 app.js 수정

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ vim app.js 

## 1.0
# const server = http.createServer((req, res) => {
#    res.statusCode = 200;
#      res.setHeader('Content-Type', 'text/plain');
#        res.end('Hello World\n');
# });

## 2.0
const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Welcome to Cloud\n');
}); 

> docker build -t node-app:0.2 .

Sending build context to Docker daemon  3.584kB
Step 1/5 : FROM node:6
 ---> 13f9ef525ab4
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> 453407bf6c80
Step 3/5 : ADD . /app
 ---> ce934d1b9504
Step 4/5 : EXPOSE 80
 ---> Running in 514c62c23ce7

Removing intermediate container 514c62c23ce7
 ---> 3698e2ca27ff
Step 5/5 : CMD ["node", "app.js"]
 ---> Running in 8f3a597c0e79
Removing intermediate container 8f3a597c0e79
 ---> ad2e00b5c075
Successfully built ad2e00b5c075
Successfully tagged node-app:0.2
```


* 컨테이너 재배포, 컨테이너 출력
	* 4000은 이미 사용중이라서, 8080으로 새롭게 실행

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker run -p 8080:80 --name my-app-2 -d node-app:0.2

> docker ps
```


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ curl http://localhost:8080
Welcome to Cloud

> curl http://localhost:4000
Hello World
```


## Debug

* 실행중인 컨테이너에서 대화식 Bash 세션을 시작하기를 원할 때, docker exec를 사용하려 실행 (다른 터미널을 열고 실행)

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker exec -it [container_id] bash

> ls

> exit # bash 세션 종료
```


* 컨테이너 메타 데이터 검사

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker inspect [container_id]
```


* 컨테이너 메타 데이터의 특정 필드 검사

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker inspect [container_id]
 
172.18.0.2
```


## 게시

* `node-app:0.2`를 `project-id`로 대체 구성

	
```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2

> docker images

> gcloud docker -- push gcr.io/[project-id]/node-app:0.2
```


* `http://gcr.io/[project-id]/node-app` 또는 `GCP 콘솔을 통해 Tools > Container Registry` 로 이동


* 컨테이너 중지, 제거 
	* `node:6` 노드 이미지를 제거하기 전에 하위 이미지 (of)를 제거해야 함


```
> cloudshell@cloudshell :~ (qwiklabs-gcp-...)$ docker stop $(docker ps -q)

> docker rm $(docker ps -aq)
> docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
> docker rmi node:6
> docker rmi $(docker images -aq) # remove remaining images
> docker images
```

















