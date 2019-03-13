2019년 Docker Study를 하며 정리한 내용이다. 각 챕터마다 참고한 원글 링크를 달아두었다.  
Docker를 직접 사용해보고 빠르게 결과를 볼 수 있는 간단한 예제들로 구성되어있다.  
학습을 위해 `MacOS` 환경에서 다룬 내용들이기 때문에 실제 서비스 운영에 바로 적용하기는 어렵다.

- [#1 Docker를 활용한 Hello World 웹 서버 만들기](#1-Docker를-활용한-Hello-World-웹-서버-만들기)
- [#2 로컬 환경에 Private Docker Registry 만들기](#2-로컬-환경에-Private-Docker-Registry-만들기)
- [#3 Docker Compose를 이용하여 모니터링 시스템 구축](#3-Docker-Compose를-이용하여-모니터링-시스템-구축)

---

# #1 Docker를 활용한 Hello World 웹 서버 만들기

> _목표:_  
> Docker 기본 개념과 명령어를 학습하고 Docker 이미지를 활용하여 웹 서버를 구축해본다.

## 참고 사이트

- Docker 공식 문서: https://docs.docker.com/get-started/
- 가장 빨리 만나는 Docker: http://pyrasis.com/book/DockerForTheReallyImpatient
- T 아카데미: https://tacademy.skplanet.com/live/player/onlineLectureDetail.action?seq=125

## Docker란?

Docker는 개발자와 시스템 관리자가 컨테이너로 개발, 배포, 실행할 수 있는 플랫폼이다.

컨테이너는 이미지를 통해 실행되는데, 이미지는 응용 프로그램을 실행하기 위해 필요한 것들(코드, 라이브러리, 환경 변수 등)을 포함하는 실행 패키지이다.

Docker 이미지를 통해 배포하면 각각 다른 서버일지라도 Docker 이미지에 세팅된 동일한 환경에서의 실행을 보장할 수 있고, Docker 이미지를 중앙에서 관리 할 수 있다.(예: Docker Hub, Docker Registry) 또한, Docker 이미지 하나로 auto scale 할 수 있고 동일한 환경에서의 테스트를 할 수 있다.

## Virtual Machine vs Container

http://pyrasis.com/book/DockerForTheReallyImpatient/Chapter01/01

- Virtual Machine: VM은 하드웨어를 소프트웨어로 가상화 한 것이다. 그래서 컴퓨터와 동일하게 VM에 OS를 설치하고, 리얼 머신에 비해 속도가 느리다. 가상화 이미지 용량도 크다.

- Container: 컨테이너는 VM 보다 경량화 된 방식으로 운영을 위한 프로그램과 라이브러리는 분리하되 OS 자원(Linux)은 호스트와 공유한다. 컨테이너는 가상화 하는 계층이 없기 때문에 메모리, 파일 시스템, 네트워크 속도가 VM에 비해 빠르고 이미지 용량이 작다.  
  (\* Docker는 Linux OS 기반이기 때문에 MacOS나 Windows OS에서는 VM위에서 Docker가 실행된다.)

## Docker 기본 명령어

#### Docker 버전 확인

```
$ docker version
```

#### Docker 컨테이너 실행

```
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

- -d: 백그라운드 모드
- -p: 호스트와 컨테이너의 포트 연결
- -v: 호스트와 컨테이너의 디렉토리 연결
- -e: 컨테이너에서 사용할 환경 변수 설정
- --name: 컨테이너 이름 설정
- --rm: 프로세스 종료 시 컨테이너 자동 제거
- -it: 터미널 입력을 위한 옵션
- --network: 네트워크 연결

컨테이너 실행 예시

```
$ docker run --rm -it ubuntu:18.04 bash

root@:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr

root@:/# cat /etc/issue
Ubuntu 18.04.1 LTS \n \l
```

#### 컨테이너 리스트 확인

ps 명령어를 입력하면 현재 실행중인 3개의 컨테이너 리스트가 나온다 .

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
446334602cb7        ubuntu:18.04        "bash"              24 minutes ago      Up 24 minutes                           suspicious_mccarthy
a32cd5426f9f        ubuntu:18.04        "bash"              26 minutes ago      Up 26 minutes                           tender_tharp
b0066176696f        ubuntu:latest       "bash"              29 minutes ago      Up 29 minutes                           dreamy_brattain

```

#### 컨테이너 종료

stop 명령어 뒤에 컨테이너 아이디를 입력하면 된다. 여러개의 컨테이너를 종료할 수 있으며, 아이디의 앞자리만 입력하여도 해당 컨테이너를 찾아 종료한다.

```
$ docker stop 446 a32 b00
446
a32
b00
```

### 이미지 삭제

```
$ docker rmi 446 a32 b00
```

## Docker로 Nginx 웹 서버 실행하기

1. 우분투 컨테이너 실행

```
$ docker run -it ubuntu:18.04 bash
```

2. 컨테이너에 Nginx 설치

```
# apt-get update
# apt-get install -y nginx
# nginx -v
nginx version: nginx/1.14.0 (Ubuntu)
```

3. Nginx 서버 동작 확인

```
# service nginx start #nginx 시작

# apt-get install -y curl
# curl -X GET localhost
```

Welcome to nginx!가 써 있는 HTML이 출력되면 성공!

4. Nginx 첫 페이지 변경

Nginx 첫 페이지에 뜨는 파일을 찾아 Welcome to nginx 텍스트를 변경해보자.

우분투 이미지에는 vi나 nano가 설치되어 있지 않다. vim을 설치해주자.

```
$ apt-get install -y vim
```

Nginx의 `/etc/nginx/sites-enabled/default` 설정 파일을 보면 root 경로를 확인할 수 있는데, 해당 위치의 html을 변경하자.

```
$ vi /var/www/html/index.nginx-debian.html
$ curl -X GET localhost
```

curl로 변경 내역이 잘 적용되었는지 확인해보자.

컨테이너가 아닌 웹 브라우저에서 접근하면 연결이 되지 않는데, 이유는 컨테이너 실행 시 포트를 연결해주지 않았기 때문이다. 포트를 연결하여 다시 실행해야한다.

이대로 컨테이너를 종료하면 변경 내역이 저장되지 않는다. nginx를 설치한 컨테이너의 이미지를 저장하고 포트를 연결하는 작업을 하자.

4. Docker 이미지 저장

컨테이너를 종료하지 말고 새로운 터미널을 열어 diff를 확인해보자.

#### Docker 이미지 변경사항 확인

```
$ docker diff [container id | name]
```

#### Docker 새로운 이미지 생성

`ubuntu:nginx`: 우분투라는 이름의 nginx 태그를 붙인 이미지 저장

```
$ docker commit [container id | name] ubuntu:nginx
```

#### 생성된 이미지 확인

nginx 태그가 달린 ubuntu 이미지가 생성되었다.

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              nginx               b72df3a79486        4 seconds ago       182MB
wordpress           latest              4771adb1849c        20 hours ago        421MB
redis               latest              0f88f9be5839        28 hours ago        95MB
mysql               5.7                 ee7cbd482336        29 hours ago        372MB


$ docker images | grep nginx
ubuntu              nginx               b72df3a79486        About a minute ago   182MB
```

#### 80, 443 포트를 연결한 nginx 컨테이너 실행

```
$ docker run -it -p 80:80 -p 443:443 ubuntu:nginx bash
$ service nginx start
```

이제 웹 브라우저에서 localhost를 접근하면 변경 사항이 저장 된 `Hello World!` 화면이 나타난다.

## Dockerfile로 이미지 만들기

실제 서비스에서는 매번 컨테이너를 변경하여 이미지를 생성할 수 없기 때문에 Dockerfile이라는 이미지 설정 파일을 이용하여 이미지를 생성한다.

위 내용을 Dockerfile로 만들어보자

```
FROM ubuntu:18.04
MAINTAINER Nara Lee <daisyleenr@gmail.com>

RUN apt-get update
RUN apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
RUN chown -R www-data:www-data /var/lib/nginx

VOLUME ["/var/www/html"]

WORKDIR /etc/nginx

CMD ["nginx"]

EXPOSE 80
EXPOSE 443
```

- FROM: 베이스 이미지
- MAINTAINER: 메인테이너 정보
- RUN: 명령 실행
- CMD: 컨테이너가 시작되었을 때 실행할 실행 파일 또는 셸 스크립트
- WORKDIR: CMD 실행 파일이 실행 될 디렉터리
- EXPOSE: 호스트와 연결할 포트

## Dockerfile로 이미지 빌드하기

```
$ docker build --tag daisyleenr/ubuntu-nginx-hello-world .
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              nginx-hello-world   9c3a92efb4db        52 seconds ago      172MB
```

생성한 이미지를 실행해보자.

```
$ docker run -d -p 80:80 -v /tmp/var/www/html:/var/www/html daisyleenr/ubuntu-nginx-hello-world
```

컨테이너 실행 후 웹 브라우저에서 localhost에 접근하면 fobbiden 에러가 나타난다. 이유는 /var/www/html을 마운트한 위치에 index.html 파일이 없기 때문인데, 해당 위치에 html을 만들어주면 화면이 잘 나타난다.

```
$ vi /tmp/var/www/html/index.html
```

## Docker Hub에 이미지 올리기

```
docker push daisyleenr/ubuntu-nginx-hello-world
```

## Docker Hub에서 이미지 받아서 웹 서버 실행하기

```
docker run -d -p 80:80 -v /tmp/var/www/html:/var/www/html daisyleenr/ubuntu-nginx-hello-world
```

---

# #2 로컬 환경에 Private Docker Registry 만들기

> _목표:_  
> Docker Registry를 이해하고 나만의 Docker Registry를 구축해본다.  
> Docker Registry 기본 개념은 공식 문서인 https://docs.docker.com/registry/ 에서 가져왔다.

## 참고 사이트

- Docker Registry 공식 문서: https://docs.docker.com/registry/

## Docker Registry란?

Registry는 Docker 이미지를 저장하고 배포할 수 있는 서버 사이드 애플리케이션이며, Apache license가 적용된 오픈 소스이다.  
보안 상 Docker 이미지를 Docker Hub에 올릴 수 없는 경우 Docker Registry를 구축하여 내부에서만 이미지를 관리할 수 있다.

또한, Registry는 인증, 권한 부여, webhook을 이용한 notification 기능을 제공한다. (이번에 다루지는 않음)

Docker Registry를 구축하기 위해서는 Docker engine 버전이 1.6.0 이상이어야 한다.

## Basic Command

1. registry 실행

```
$ docker run -d -p 5000:5000 --name registry registry:2
```

2. 샘플로 사용할 ubuntu 이미지를 docker hub에서 pull 한다.

```
$ docker pull ubuntu
```

3. 이미지를 registry를 업로드 할 수 있도록 tag를 지정한다.

```
$ docker image tag ubuntu localhost:5000/myfirstimage
$ docker images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
localhost:5000/myfirstimage           latest              94e814e2efa8        6 hours ago         88.9MB
ubuntu                                latest              94e814e2efa8        6 hours ago         88.9MB
```

4. 이미지를 push 한다.

```
$ docker push localhost:5000/myfirstimage
The push refers to repository [localhost:5000/myfirstimage]
b57c79f4a9f3: Pushed
d60e01b37e74: Pushed
e45cfbc98a50: Pushed
762d8e1a6054: Pushing [=========================================>         ]  74.13MB/88.91MB

$ curl -X GET http://localhost:5000/v2/_catalog
{"repositories":["myfirstimage"]}
```

5. 이미지를 pull 한다.

```
$ docker pull localhost:5000/myfirstimage
Using default tag: latest
latest: Pulling from myfirstimage
Digest: sha256:f2557f94cac1cc4509d0483cb6e302da841ecd6f82eb2e91dc7ba6cfd0c580ab
Status: Image is up to date for localhost:5000/myfirstimage:latest
```

6. registry를 종료하고 데이터를 삭제한다.

```
$ docker container stop registry && docker container rm -v registry
```

- 주의) 기본 예제는 테스트용이므로 실제 프로덕션에 구축할 때에는 아래 문서를 참고한다.
  https://docs.docker.com/registry/configuration/

## Docker 이미지 이름에 대한 이해

Docker command에 사용되는 이미지 이름은 이미지의 소스를 가리킨다.  
`docker pull ubuntu`의 경우 `docker pull docker.io/library/ubuntu`가 간소화 된 것 이다.  
`docker pull myregistrydomain:port/foo/bar`는 `myregistrydomain:port` registry의 `foo/bar`이미지를 찾는다.

## Use cases

- CI/CD 구축  
  `git commit` - `build(CI)` - `push a new image to your Registry` - `notification from Registry` - `trigger` - `deploy`
- large cluster of machines 환경에서 빠르게 이미지 배포할 때 사용
- isolated network 환경에서 배포할 때 사용

## Requirement

Registry를 시작하는 것은 쉽지만, 프로덕션 환경에서 운영하려면 다른 서비스처럼 운영 기술이 필요하다. availability, scalability, logging, log processing, systems monitoring, security등에 익숙해야 한다. http 및 네트워크에 대한 이해 및 golang 지식이 있으면 advanced operations or hacking에 유용하다. (Registry v2가 golang으로 개발되었음)

## Basic configuration

https://docs.docker.com/registry/deploying/#basic-configuration

## Storage customization

Registry는 기본적으로 docker volumn에 파일을 저장한다.  
특정 위치에 저장하고 싶은 경우 -v 옵션으로 설정한다. (-v <호스트 디렉터리>:<컨테이너 디렉터리>)

```
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /Users/naralee/workspace/git/docker-study/example/registry:/var/lib/registry \
  registry:2
```

또한, Amazon S3 등 클라우드 스토리지에 저장할 수도 있다. 자세한 내용은 아래 링크 참고.  
https://docs.docker.com/registry/configuration/#storage

## 보안 관련

localhost가 아닌 외부에서 접근 가능하게 하려면 TLS(SSL)를 사용하여 Registry를 보호해야 한다.  
Run an externally-accessible registry, Load balancing considerations, Restricting access는 이번 스터디에서는 생략한다.

## Docker Registry Web UI

Docker Registry를 Web UI로 보여주는 다양한 오픈소스들이 존재한다. (아래 링크 참고)  
https://github.com/veggiemonk/awesome-docker/#web

이 중에서 star가 가장 많은 atcol/docker-registry-ui를 적용해보았지만, 이미지 리스트가 뜨지 않았고 관련 이슈가 등록이 되어있는데 반영이 되지 않는 듯 하여 다른 프로젝트로 변경한다.  
https://github.com/atcol/docker-registry-ui/issues/170

port.us(https://github.com/SUSE/Portus) 등 registry의 다양한 기능을 Web으로 이용할 수 있는 프로젝트들이 있는데, 나는 가장 단순하게 등록된 이미지의 정보만 필요해서 아래의 프로젝트를 선택하였다.  
https://hub.docker.com/r/klausmeyer/docker-registry-browser/

```
$ docker run --name registry-browser -d -p 8080:8080 -e DOCKER_REGISTRY_URL=http://192.168.0.3:5000 klausmeyer/docker-registry-browser
```

localhost:8080을 접속하면 아래 화면에 나의 docker registry에 올라간 이미지 목록이 나온다.

<img width="726" alt="스크린샷 2019-03-13 오후 6 21 29" src="https://user-images.githubusercontent.com/12470452/54267684-0403bc80-45bd-11e9-9799-27d010c04c15.png">

## Registry를 운영하게 될 때 추가로 스터디 할 내용들

- 보안: https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry
- 실제 운영에서 적용해야하는 configuration: https://docs.docker.com/registry/configuration/
- notifications: https://docs.docker.com/registry/notifications/

---

# #3 Docker Compose를 이용하여 모니터링 시스템 구축

> _목표:_  
> Docker Compose를 학습하기 위해 시스템 지표를 수집하는 간단한 모니터링 시스템을 구축한다.  
> 모니터링 시스템은 오픈소스인 Telegraf + InfluxDB + Grafana를 Docker Compose를 이용하여 구축한다.

## 참고 사이트

- Docker, Telegraf, Influxdb, Grafana로 5분만에 system metrics 수집하기:  
  https://towardsdatascience.com/get-system-metrics-for-5-min-with-docker-telegraf-influxdb-and-grafana-97cfd957f0ac

## Docker Compose란?

여러 컨테이너를 정의하고 실행하기 위한 도구이다. YAML 파일에 서비스에 필요한 컨테이너 설정을 정의하고 single command로 구성 할 수 있다.

## InfluxDB, Grafana 구축을 위한 docker-compose.yml 파일 생성

프로젝트 폴더를 생성하고 docker-compose.yml을 작성한다.

```
$ mkdir /opt/monitoring && cd /opt/monitoring
$ vi docker-compose.yml
```

docker-compose.yml

```
version: "2"
services:
  grafana: # 서비스 이름
    image: grafana/grafana # 사용할 이미지
    container_name: grafana # 컨테이너 이름
    restart: always # command 실행 결과에 따라 재시작한다
    ports:
      - 3000:3000 # port 설정
    networks:
      - monitoring # 네트워크를 생성하고 컨테이너를 연결시키면 해당 네트워크 안에 속한 컨테이너끼리 서로 접속할 수 있다
    volumes:
      - grafana-volume:/var/lib/grafana
  influxdb:
    image: influxdb
    container_name: influxdb
    restart: always
    ports:
      - 8086:8086
    networks:
      - monitoring
    volumes:
      - influxdb-volume:/var/lib/influxdb
networks:
  monitoring:
volumes:
  grafana-volume:
    external: true
  influxdb-volume:
    external: true # 프로젝트를 생성할 때마다 볼륨을 새로 만들지 않고 기존 볼륨을 사용한다
```

## Docker network와 volume 생성

```
$ docker network create monitoring
$ docker volume create grafana-volume
$ docker volume create influxdb-volume

$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
f5ecb2af5e63        bridge              bridge              local
754450c648ac        host                host                local
2a2bf1d1fd43        monitoring          bridge              local
cc9ffeab029c        none                null                local

$ docker volume ls
local
DRIVER              VOLUME NAME
local               grafana-volume
local               influxdb-volume
```

## InfluxDB 환경 설정

InfluxDB database와 user를 생성하기 위해 한번은 influxDB parameter를 넣고 컨테이너를 실행해야 한다. --rm 옵션을 주고 컨테이너를 실행하면 환경 설정을 influxdb-volume에 구성하고 컨테이너를 삭제한다.

```
$ docker run --rm \
  -e INFLUXDB_DB=telegraf -e INFLUXDB_ADMIN_ENABLED=true \
  -e INFLUXDB_ADMIN_USER=admin \
  -e INFLUXDB_ADMIN_PASSWORD=supersecretpassword \
  -e INFLUXDB_USER=telegraf -e INFLUXDB_USER_PASSWORD=secretpassword \
  -v influxdb-volume:/var/lib/influxdb \
  influxdb /init-influxdb.sh
```

## docker-compose 실행

```
docker-compose up -d
```

## Grafana 설정

1. localhost:3000 접속 후 로그인한다. (admin/admin)
   <img width="1669" alt="스크린샷 2019-03-13 오후 6 59 25" src="https://user-images.githubusercontent.com/12470452/54270186-2d731700-45c2-11e9-819c-30d03bc24e91.png">

2. Add data source
   ![image](https://user-images.githubusercontent.com/12470452/54270418-bbe79880-45c2-11e9-8f32-e8caf2429a22.png)  
   ![image](https://user-images.githubusercontent.com/12470452/54270622-28fb2e00-45c3-11e9-9410-7743bac09b00.png)  
   입력해줘야 하는 것들 (--rm 옵션으로 설정했던 환경변수 값)

- HTTP URL: http://influxdb:8086
- Database: telegraf
- User: telegraf
- Password: secretpassword

3. Grafana 대시보드 템플릿 추가  
   https://grafana.com/dashboards/914  
   ![image](https://user-images.githubusercontent.com/12470452/54270902-c191ae00-45c3-11e9-84c9-1908e7c99133.png)

4. Telegraf 설치  
   https://docs.influxdata.com/telegraf/v1.9/introduction/installation/#installation  
   ![image](https://user-images.githubusercontent.com/12470452/54270951-dbcb8c00-45c3-11e9-8702-a40efecd4ebd.png)

## Telegraf + InfluxDB + Grafana

- Telegraf:
- InfluxDB:
- Grafana:
