# docker-study

Docker 기본 개념과 명령어, 컨테이너 배포까지 정리한 내용입니다.

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
