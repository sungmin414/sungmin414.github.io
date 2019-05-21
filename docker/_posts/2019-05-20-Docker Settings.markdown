---
layout: post
title: "[Docker] Docker Settings"
categories: posts
tags: docker
---

# [Docker] Settings 기본 명령어

## Docker Settings, 기본 명령어

+ [Docker hub](https://id.docker.com)
	+ docker hub에 들어가서 가입하고 docker 받기

+ [Docker 문서 (mac)](https://docs.docker.com/docker-for-mac/)

	+ 버전 확인, 에플리케이션 탐색, 환경 설정 메뉴 등등 


## 컨테이너 실행하기

+ 도커 실행하는 명령어

>`docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]`

### 자주 사용하는 옵션 

옵션| 설명 |
---| --- |
-d | detached mode 백그라운드 모드|
-p | 호스트와 컨테이너의 포트를 연결 (포워딩)
-v | 호스트와 컨테이너의 디렉토리를 연결 (마운트)
-e | 컨테이너 내에서 사용할 환경변수 설정
-name | 컨테이너 이름 설정
-rm | 프로세스 종료시 컨테이너 자동 제거
-it | -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
-link | 컨테이너 연결[컨테이너명:별칭]


+ 컨테이너 내부 들어가기
	+ `docker run ubuntu:18.04`

> run 명령어를 사용하면 사용할 이미지가 저장되어 있는지 확인하고 없다면 다운로드 한 후 컨테이너를 생성하고 시작 함

+ `docker run --rm -it ubuntu:18.04 /bin/bash`
	+ 컨테이너 내부에 들어가기위해 `bash` 쉘을 실행하고 키보드 입력을 위해 `-it` 옵션을 주고 `--rm`은  프로세스가 종료되면 컨테이너가 자동으로 삭제되도록 함
	+ `apt update` 패키지 업데이트

## 도커 기본 명령어

### 컨테이너 목록 확인하는 명령어

> `docker ps [OPTIONS]`

옵션 | 설명 |
--- | --- |
ps | 실행중인 컨테이너 목록을 보여줌
-a | 처음 실행했다가 종료된 컨테이너가 추가로 보임, 컨테이너는 종료되어도 삭제되지 않고 남아있음. 종료된 건 다시 시작할 수 있고 컨테이너의 읽기/쓰기 레이어는 그대로 존재함. 명시적으로 삭제를 하면 깔끔하게 컨테이너가 제거된다.

### 컨테이너 중지하기 (stop)

+ 옵션은 특별한게 없고 실행중인 컨테이너를 하나 또는 여러개 (띄어쓰기로 구분)중지 할 수 있음.

	> `docker stop [OPTIONS] CONTAINER [CONTAINER...]`
	


### 컨테이너 제거하기 (rm)

+ 특별한 옵션은 없음, 종료된 컨테이너를 하나 또는 여러개 삭제
	> `docker rm [OPTIONS] CONTAINER [CONTAINER...]`



+ 중지된 컨테이너 ID를 가져와서 한번에 삭제하기
	> `docker rm -v $(docker ps -a -q -f status=exited)`


### 이미지 목록 확인 (images)

> `docker images [OPTIONS] [REPOSITORY[:TAG]]`

+ 도커 이미지 목록 확인

	> `docker images`
	+ 이미지 주소, 태그, ID, 생성지점, 용량보여줌
	+ 이미지가 너무 많이 쌓이면 용량을 차지하기 때문에 사용하지 않는 이미지는 지우는 것이 좋음

+ 사용하지 않는 이미지 지우기
	> `docker system prune`

### 이미지 다운로드하기 (pull)

> `docker pull [OPTIONS] NAME[:TAG|@DIGEST]`

+ `docker pull ubuntu:18.04`
	+ `run` 명령어를 입력하면 이미지가 없을때 자동으로 다운받음
	+ `pull` 명령어는 최신버전으로 다시 다운 받음(이미지가 업데이트 된경우에 pull 명령어로 새로 다운받을 수 있음) 

### 이미지 삭제하기 (rmi)

> `docker rmi [OPTIONS] IMAGES [IMAGE...]`

+ `images` 명령어를 통해 얻은 이미지 목록에서 이미지 ID를 입력하면 삭제 된다.
+ 컨테이너가 실행중인 이미지는 삭제되지 않음

```
#예)

docker images 	# get image ID
docker rmi ${IMAGE_ID}
```

### 컨테이너 로그 보기 (log)

+ 컨테이너가 정상적으로 동작하는지 확인하는 방법
	+ `기본 옵션과, -f, --tail 옵션`
	
	> `docker logs [OPTIONS] CONTAINER`

```
예)

docker ps
docker logs ${CONTAINER_ID}
```

+ `docker logs --tail 10 ${CONTAINER_ID}`
	+ `--tail`옵션으로 마지막 10줄만 출력
+ `docker logs -f ${CONTAINER_ID}`
	+ 실시간으로 로그 확인
	+ `ctrl + c` 로그 보기 중지

### 컨테이너 명령어 실행하기 (exec)

> `docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`

+ `exec`는 실행중인 컨테이너에 명령어를 내리는 정도

### 컨테이너 업데이트

+ 도커에서 컨테이너를 업데이트 하려면 새버전의 이미지를 다운 (pull)받고 기존 컨테이너를 삭제(stop, rm)한 후 새 이미지를 기반으로 새컨테이너를 실행(run)

### Data volumes

+ 컨테이너를 삭제한다면 데이터베이스에 쌓였던 데이터가 모두 사라진다는 것이고 웹 어프리케이션이라면 그동안 자용자가 업로드한 이미지가 모두 사라진다
+ AWS S3같은 클라우드 서비스를 이용해서 데이터 볼륨을 컨테이너에 추가해서 사용
	+ 데이터 볼륨을 사용하면 해당 디렉토리는 컨테이너와 별도로 저장되고 컨테이너를 삭제시 데이터가 지워지지 않음

> 호스트의 디렉토리를 마운트해서 사용하는 방법(`run`명령어 옵션 `-v`사용하기)

```
예)

# before
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  mysql:5.7

# after
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  -v /my/own/datadir:/var/lib/mysql \ 			# <- volume mount
  mysql:5.7
```

+ 위 샘플은 호스트의 `/my/own/datadir`디렉토리를 컨테이너의 `/var/lib/mysql`디렉토리로 마운트 하였습니다. 이제 데이터베이스 파일은 호스트의 `/my/own/datadir`디렉토리에 저장되고 컨테이너를 삭제해도 데이터는 사라지지 않습니다. 최신버전의 MySQL 이미지를 다운받고 다시 컨테이너를 실행할 때 동일한 디렉토리를 마운트 한다면 그대로 데이터를 사용할 수 있습니다


### Docker Compose

+ 복잡한 설정을 쉽게 관리하기 위해 YAML방식의 설정파일을 이용한 Docker Compose툴을 제공



## 도커 이미지 만들기

+ 도커는 이미지를 만들기 위해 `Dockerfile`이라는 이미지 빌드용 DSL(Domain Specific Language) 파일을 사용한다. 단순 텍스트 파일로 일반적으로 소스와 함께 관리함.

+ 도커 빌드 중엔 키보드를 입력할 수 없기 때문에 (y/n)을 물어보는 걸 방지하기 위해 -y 옵션을 추가한다.

+ `Gemfile` 패키지 관리
+ 이미지를 빌드하는 명령어
	> `docker build [OPTIONS] PATH | URL | -`

+ 생성할 이미지 이름을 지정하기 위한 `-t(--tag)`옵션만 알면 빌드가 가능
	+ 예) `docker build -t [images_name] -f Dockerfile .`

+ 이미지 생성후 잘동작하는지 컨테이너 실행하기(웹서버 실행해보기)
	+ `docker run --rm -it -p 7000:8000 images_name`


옵션 | 사용법 | 설명 |
--- | ---- | --- |
FROM | FROM [image]:[tag], FROM ubuntu:18.04 | 베이스 이미지 지정, 반드시 지정해야 하며 어떤 이미지도 베이스 이미지가 될수 있다.
MAINTAINER | MAINTAINER [name], MAINTAINER xxxx@xxxx.com | Dockerfile을 관리하는 사람의 이름 또는 이메일 정보를 적는다, 빌드에 영향을 주지는 않음
COPY | COPY [src]...[dest], COPY ./  /srv/project/ | 파일이나 디렉토리를 이미지로 복사, 일반적으로 소스를 복사하는데 사용, `target`디렉토리가 없다면 자동으로 생성
ADD | ADD [src]...[dest], ADD ./  /srv/project/ | `COPY`명령어와 매우 유사하나 몇가지 추가 기능이 있음, `src`에 파일 대신 URL을 입력 할 수 있고 `src`에 압축 파일을 입력하는 경우 자동으로 압축을 해제하면서 복사함
RUN | RUN [command], RUN bundle install | 가장 많이 사용하는 구문, 명령어를 그대로 실행, 내부적으로 `/bin/sh -c`뒤에 명령어를 실행하는 방식
CMD | CMD python3 manage.py runserver 0:8000 | 도커 컨테이너가 실행 되었을 때 실행되는 명령어를 정의, 빌드할 때는 실행되지 않으며 여러개의 `CMD`가 존재할 경우 가장 마지막 `CMD`만 실행된다,
WORKDIR | WORKDIR /srv/project | RUN, CMD, ADD, COPY,등이 이루어질 기본 디렉토리를 설정, 각 명령어의 현재 디렉토리는 한 줄 한 줄마다 초기화되기 때문에 같은 디렉토리에서 계속 작업하기 위해서 `WORKDIR`을 사용
EXPOSE | EXPOSE [port] [<port>...], EXPOSE 8000 | 도커 컨테이너가 실행 되었을 때 요청을 기다리고 있는 (Listen)포트를 지정, 여러개의 포트를 지정할 수 있다
VOLUME | VOLUME ["/data"] | 컨테이너 외부에 파일시스템을 마운트 할 때 사용, 반드시 지정하지 않아도 마운트 할 수 있지만 기본적으로 지정하는 것이 좋음
ENV | ENV [key][value], ENV DB_URL mysql | 컨테이너에서 사용할 환경변수를 지정, 컨테이너를 실행할 때 `-e`옵션을 사용하면 기존 값을 오버라이딩 하게된다.


+ 위표는 가장 많이 사용하는 명령어 다른명령어가 필요할때 아래 문서보기
	+ [Dockerfile 공식문서](https://docs.docker.com/engine/reference/builder/)

```
# Dockerfile 설정

# 이미지 빌드(ec2-deploy폴더에서 실행)
# docker build -t ec2-deploy -f Dockerfile .
FROM                ubuntu:18.04
MAINTAINER          psungmin88.1@gmail.com

# 패키지 업그레이드, python3설치
RUN                 apt -y update
RUN                 apt -y dist-upgrade
RUN                 apt -y install python3-pip


# Nginx, uWSGI 설치 (WebServer, WSGI)
RUN                 apt -y install nginx supervisor
RUN                 pip3 install uwsgi

# requirements.txt 파일만 복사 후, 패키지 설치
# requirements.txt 파일의 내용이 바뀌지 않으면 pip3 install ...부분이 재실행되지 않음
COPY                requirements.txt /tmp/
RUN                 pip3 install -r /tmp/requirements.txt

# 전체 소스코드 복사
COPY                ./  /srv/project/
WORKDIR             /srv/project

# settings모듈에 대한 환경변수 설정
ENV                 DJANGO_SETTINGS_MODULE  config.settings.production

# 프로세스를 실행할 명령
WORKDIR             /srv/project/app
RUN                 python3 manage.py collectstatic --noinput

# Nginx
# 기존에 존재하던 Nginx설정파일들 삭제
RUN                 rm -rf  /etc/nginx/sites-available/*
RUN                 rm -rf  /etc/nginx/sites-enabled/*

# 프로젝트 Nginx설정파일 복사 및 enabled로 링크 설정
RUN                 cp -f   /srv/project/.config/app.nginx \
                            /etc/nginx/sites-available/
RUN                 ln -sf  /etc/nginx/sites-available/app.nginx \
                            /etc/nginx/sites-enabled/app.nginx

# supervisor설정파일 복사
RUN                 cp -f   /srv/project/.config/supervisor.conf \
                            /etc/supervisor/conf.d/

# Command로 supervisor실행
CMD                 supervisord -n

```


### uWSGI 정상 동작 확인

```
uwsgi \
--http :(port) \
--home (virtualenv경로) \
--chdir <django프로젝트 경로> \
-w 설정파일.wsgi

---------------------------------

uwsgi \
--http :8000 \
--chdir /srv/project/app \
--wsgi config.wsgi
```

### settings.py (패키지로 나누기)

+ dev, production 개발용 배포용 나누기
	+ `export DJANGO_SETTINGS_MODULE=config.settings.dev`
	+ `export DJANGO_SETTINGS_MODULE=config.settings.production`

### 정적 static 설정 (SATATIC_ROOT)

+ `STATIC_ROOT = os.path.join(ROOT_DIR, '.static')`
	
	> `./manage.py collectstatic`


### Uwsgi, Nginx 설정

+ python 플러그인(nginx support 설치)

+ `.config -> uwsgi.ini, app.nginx 파일생성`

```
# uwsgi.ini
[uwsgi]
; 파이썬 애플리케이션의 경로 (우리의 경우엔 Django project)
chdir = /srv/project/app
;application과 UWSGI를 연결해주는 모듈
wsgi = config.wsgi
; socket을 사용해 연결을 주고받음
socket = /tmp/app.sock
; uWSGI가 종료되면 자동으로 소켓 파일을 삭제
vacuum = true
; socket파일의 소유자 변
chown-socket = www-data

```

```
# app.nginx

# /etc/nginx/sites-available/
server {
    # 80번 포트로부터 받은 요청을 처리
    listen 80;

    # 도메인이 'localhost'에 해당 할 때
    server_name localhost;

    # 인코딩방식
    charset utf-8;

    # request/repseponse의 최대 사이즈
    # (기본값이 매우 작음)
    client_max_body_size 128M;

    # ('/'부터 시작) -> 모든 URL연결에 대해
    location / {
        # uwsgi와의 연결에 unix소켓을 사용
        # "/tmp/app.sock" 파일을 사용
        uwsgi_pass  unix:///tmp/app.sock;
        include     uwsgi_params;
    }

    location /static/ {
        alias /srv/project/.static/;
    }
    location /media/ {
        alias /srv/proejct/.media/;
    }
}
```

### supervisor 설정
+ 여러 프로세스를 모니터하고 제어할수 있음

```
# .config/supervisor.conf

[program:nginx]
command=nginx -g 'daemon off;'

[program:uwsgi]
command=uwsgi --ini /srv/project/.config/uwsgi.ini


```
	

