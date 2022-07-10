# jenkins整合sonarqube

## 1.docker 安装

```sh
yum install docker
systemctl  start docker
docker ps
```



## 2. jenkins安装

```sh
docker search jenkins
#打开jenkin官网https://www.jenkins.io/doc/book/installing/docker/
[root@hadoop101 ~]# docker network create jenkins
8df10ac28f315b01302a91e68e73b573f61a0715740b27d6044ed37dc39430b6
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
  
docker stop jenkins-docker
```



## 3. sonarqube安装

```sh
 docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```

