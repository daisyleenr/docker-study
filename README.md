# Docker Study #1 Docker 이미지를 활용한 웹 서버 구축

> Docker Study #1 summary:  
> Docker 기본 개념과 명령어를 학습하고 Docker 이미지를 활용하여 웹 서버 구축

## 참고 사이트

- Docker 공식 문서: https://docs.docker.com/get-started/
- T 아카데미: https://tacademy.skplanet.com/live/player/onlineLectureDetail.action?seq=125
- 가장 빨리 만나는 Docker: http://pyrasis.com/book/DockerForTheReallyImpatient

## Docker란

`Immutable Infrastructure`는 호스트 OS와 서비스 운영 환경을 분리하고, 한 번 설정한 운영 환경은 변경하지 않는다(Immutable)는 개념이다.

Docker는 OS와 서비스 운영 환경을 분리하고 서비스 운영 환경을 이미지로 생성할 수 있다.

Docker 이미지를 통해 배포하면 동일한 환경에서의 실행을 보장할 수 있고 이미지를 중앙에서 배포와 관리를 할 수 있다. 이미지 하나로 계속 서비스를 확장할 수 있고(auto scale) 동일한 환경에서의 테스트를 가능하게 한다.

## Virtual Machine vs Container

http://pyrasis.com/book/DockerForTheReallyImpatient/Chapter01/01

- Virtual Machine: VM은 하드웨어를 소프트웨어로 가상화 한 것이다. 그래서 컴퓨터와 동일하게 그 위에 OS를 설치하고 리얼 머신에 비해 속도가 느리다. 가상화 이미지도 역시 용량이 크다

- Container: 컨테이너는 VM 보다 경량화 된 방식으로 운영을 위한 프로그램과 라이브러리마 격리하고 OS 자원은 호스트와 공유한다. 가상화 하는 계층이 없기 때문에 메모리, 파일 시스템, 네트워크 속도가 VM에 비해 빠르고 용량이 작다.
  Docker는 리눅스 OS 기반이기 때문에 MacOS나 Windows OS OS에서는 VM위에 Docker가 실행되는 방식이다.

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

# Docker Study #2 Private Docker Registry 구축과 Dockerfile 및 Docker Compose 익히기

> Docker Study #2 summary:  
> 나만의 Docker Registry를 구축하고 Dockerfile과 Docker Compose 예제를 만들어봅니다.  
> Docker Registry 기본 개념은 공식 문서인 https://docs.docker.com/registry/ 의 내용을 가져왔습니다.

## 참고 사이트

- Docker Registry 공식 문서: https://docs.docker.com/registry/

## Docker Registry란?

Registry는 Docker 이미지를 저장하고 배포할 수 있는 서버 사이드 애플리케이션이며, Apache license가 적용된 오픈 소스이다.  
보안 상 Docker 이미지를 Docker Hub에 올릴 수 없는 경우 Docker Private Registry를 구축하여 내부에서만 이미지를 관리할 수 있다.

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
특정 위치에 저장하고 싶은 경우 -v 옵션으로 설정한다.

```
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /mnt/registry:/var/lib/registry \
  registry:2
```

또한, Amazon S3 등 클라우드 스토리지에 저장할 수도 있다. 자세한 내용은 아래 링크 참고.  
https://docs.docker.com/registry/configuration/#storage

## 보안 관련

localhost가 아닌 외부에서 접근 가능하게 하려면 TLS(SSL)를 사용하여 Registry를 보호해야 한다.  
Run an externally-accessible registry, Load balancing considerations, Restricting access는 이번 스터디에서는 생략한다.

## Registry를 운영하게 될 때 추가로 스터디 할 내용들

- 보안: https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry
- 실제 운영에서 적용해야하는 configuration: https://docs.docker.com/registry/configuration/
- notifications: https://docs.docker.com/registry/notifications/
