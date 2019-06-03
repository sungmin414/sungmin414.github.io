---
layout: post
title: "[AWS] EB[Elastic Beanstalk]?"
categories: posts
tags: aws
---

# [AWS]EB(Elastic Beanstalk)

## EB(Elastic Beanstalk)

+ [EB문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html)

## EB CLI(Command Line Interface) 사용법

+ EB CLI 설치
	
	> `pip install awsebcli`	(가상환경 안에 설치)
	
+ 사용 순서
	
```
서비스 (IAM 검색) -> 사용자추가(사용자 이름, 프로그래밍 방식 액세스 체크) 

-> 기존 정책 직접 연결(elasticbeanstalk, elasticloadbalancing 검색-> 
AWSElasticBeanstalkFullAccess, ElasticLoadBalancingFullAccess 체크)

-> 사용자 만들기 (access_key_id, access_key(기록))

```

+ ElasticLoadBalancingFullAccess
	
	> Domain -> DNS -> ELB(Elastic Load Balancing) -> EC2,(ELB는 여러개의 EC2의 같은 목록을 만들어줌 한곳에 데이터를 넘기면 cpu사용량이 넘치면 느려지고 서버가 다운대기 때문에 cpu사용량이 넘치면 같은 EC2목록을 만들어 넘겨줌) 

## EB CLI 구성

+ 사용법

	> `eb init --profile eb` eb는 credentials 안에 [eb] [ ]괄호 안에 이름 
	
```
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) cn-northwest-1 : China (Ningxia)
14) us-east-2 : US East (Ohio)
15) ca-central-1 : Canada (Central)
16) eu-west-2 : EU (London)
17) eu-west-3 : EU (Paris)
18) eu-north-1 : EU (Stockholm)
(default is 3): 10

Enter Application Name
(default is "ec2_deploy"): EB Docker Deploy
Application EB Docker Deploy has been created.

It appears you are using Docker. Is this correct?
(Y/n):
Do you wish to continue with CodeCommit? (y/N) (default is n):
Do you want to set up SSH for your instances?
(Y/n):

Select a keypair.
1) psm_key
2) [ Create new KeyPair ]
(default is 1): 2

# 키페어를 새로 만들때 비밀번호를 만들어서 사용할수 있음.
# 단점은 비밀번호를 계속쳐줘야하고 
# 장점은 키페어를 잃어버려도 비밀번호를 알지못하면 남이 사용못함
```

## 환경 생성하기 (eb create)

> `eb create --profile eb` 

```
Enter Environment Name
(default is EBDockerDeploy-dev): eb-docker-deploy-master
Enter DNS CNAME prefix
(default is eb-docker-deploy-master2): eb-docker-deploy-master
That cname is not available. Please choose another.
Enter DNS CNAME prefix
(default is eb-docker-deploy-master): psm-eb-docker-deploy-master

Select a load balancer type
1) classic
2) application
3) network
(default is 2): 2

```

### EC2 인스턴스가 2개 켜져있으면 과금이 들기 때문에 사용하지 안하는 것은 종료시키기

## Error가 날경우 확인

> `eb ssh`

```
# log 경로

> vi /var/log/eb-activity.log

```

+ 오류 확인후 뭐가 잘못되었는지 확인하고 수정
	+ docker build 해보고 처음에 확인해보기 잘 실행되는지( build가 안되서 오류날때가 많음 )

### Dockerfile 용량 줄이기

+ python이 용량을 많이 찾이하기 때문에 image로 설치하기
	+ [image_python설치](https://hub.docker.com/_/python)
	+ python slim(python중에서 최소기능만 사용해서 용량이적음)

> `docker pull python:3.6.8-slim`

+ python 설치후 확인
	+ `docker run --rm -it python:3.6.8-slim /bin/bash`
	+ 사용할 패키지들이 설치 되나 확인후 Dockerfile수정

### wsgi.py 패키지화

```
> dev.py

import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.dev')

application = get_wsgi_application()
```

```
> production.py

import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.production')

application = get_wsgi_application()
```

```
> settings/dev.py, production.py

WSGI_APPLICATION = 'config.wsgi.dev.application'

WSGI_APPLICATION = 'config.wsgi.production.application'
```
```
> .config/uwsgi.ini

wsgi = config.wsgi.production
```

+ Dockerfile 나누기
	+ docker build를 할때 안해도 될 반복적인 일을하기떄문에 시간적문제와 실용성을 위해서 나눔

```
> Dockerfile

# docker build -t eb_deploy -f Dockerfile .
FROM                eb_docker:base
ENV                 DJANGO_SETTINGS_MODULE config.settings.production
# 전체 소스코드 복사
COPY                ./  /srv/project/
WORKDIR             /srv/project


# 프로세스를 실행할 명령
WORKDIR             /srv/project/app
RUN                 python3 manage.py collectstatic --noinput

# Nginx
RUN                 rm -rf  /etc/nginx/sites-available/* && \
                    rm -rf  /etc/nginx/sites-enabled/* && \
# 프로젝트 Nginx설정파일 복사 및 enabled로 링크 설정
                    cp -f   /srv/project/.config/app.nginx \
                            /etc/nginx/sites-available/ && \
                    ln -sf  /etc/nginx/sites-available/app.nginx \
                            /etc/nginx/sites-enabled/app.nginx

# supervisor설정파일 복사
RUN                 cp -f   /srv/project/.config/supervisor.conf \
                            /etc/supervisor/conf.d/

# 80번 포트 개방
EXPOSE              80

# Command로 supervisor실행
CMD                 supervisord -n

```

```
> Dockerfile.base

# docker build -t eb_deploy:base -f Dockerfile.base .
FROM                python:3.6.8-slim
MAINTAINER          psungmin88.1@gmail.com

# settings모듈에 대한 환경변수 설정
ENV                 LANG                    C.UTF-8

# 패키지 업그레이드, gcc, nginx, supervisor, uwsgi 설치
RUN                 apt -y update
RUN                 apt -y dist-upgrade
RUN                 apt -y install gcc nginx supervisor
RUN                 pip3 install uwsgi

# requirements.txt 파일만 복사 후, 패키지 설치
# requirements.txt 파일의 내용이 바뀌지 않으면 pip3 install ...부분이 재실행되지 않음
COPY                requirements-production.txt /tmp/requirements.txt
RUN                 pip3 install -r /tmp/requirements.txt

```

+ `docker build` 해주기

+ 환경 버전을 업데이트 할때 deploy사용
	+ `eb deploy --profile eb`

### Docker hub

+ Dockerfile에 python:3.6.8-slim을 외부에서 받았기때문에 local에 python이 없기떄문에 실행이 안되므로 docker hub에 images를 올려줘야 정상 작동함

+ [docker hub](https://cloud.docker.com)

```
> 사용법

Repositories 클릭 -> Create Repository 클릭 

-> Repository_NAME적기, Public체크 하고 만들기

```

+ 이미 있는 이미지에 테그 걸기
	+ `docker tag eb_docker:base dockerhub_계정명/eb_docker:base`
+ docker login
	+ `docker login`
+ docker push
	+ `docker push dockerhub_계정명/images_repository:tagname`
+ 배포할때 항상 git commit 하고 deploy 하기
	+ 강제로 커밋하기(`git add -f 파일명`)

> 이렇게 시도했을경우 나중에 json 파일이 없다고 오류날것임(json파일은 올리면 안됨)
> 이때 커밋하지 않고 환경에 배포하려면 --staged 옵션을 사용하여 스테이징 영역에 추가된 변경 사항을 배포할 수 있다.

+ `eb deploy --profile eb --staged`

> 만약 배포가 완성 되었다면 commit 한 json파일을 지우기

+ `git reset HEAD 파일명`

### deploy script 작성

> 위 내용처럼 staged를 사용해서 json을 사용하고 git reset을 사용해서 json을 지울수는 있지만 사람마다 실수해서 json파일을 올리수도 있기때문에 밑에처럼 설정해서 사용하기

```
> root/deploy.sh

#!/usr/bin/env bash
git add -f .secrets/
eb deploy --profile eb --staged
git reset HEAD .secrets/

```

+ 위 설정한 값을 실행할때는 `sh deploy.sh`로 실행하는데 그냥 쓰고싶을때는 권한을 바꿔준다(`chmod 755 deploy.sh`)
	+ 권한을 바꾼 후 사용법 `./deploy.sh`로 실행 하면된다.


### Django logging

+ 실행중에 발생하는 이벤트를 기록하는 파일 
	+ [logging문서](https://docs.djangoproject.com/en/2.2/topics/logging/)
	
```
> settings/production.py

# 이 로그 코드를 production에 사용하기전 dev에서 먼저 테스트해보고 사용하기
# 로그폴더 생성
LOG_DIR = os.path.join(ROOT_DIR, '.log')
if not os.path.exists(LOG_DIR):
    os.makedirs(LOG_DIR, exist_ok=True)

LOGGING = {
    'version': 1,
    'formatters': {
        'default': {
            'format': '[%(levelname)s] %(name)s (%(asctime)s)\n\t%(message)s'
        },
    },
    'handlers': {
        'file_error': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'ERROR',
            'filename': os.path.join(LOG_DIR, 'django.log'),
            'formatter': 'default',
            'maxBytes': 1048576,
            'backupCount': 10,
        },
        'file_info': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'ERROR',
            'filename': os.path.join(LOG_DIR, 'django.log'),
            'formatter': 'default',
            'maxBytes': 1048576,
            'backupCount': 10,
        },
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
        }
    },
    'loggers': {
        'django': {
            'handlers': ['file_error', 'file_info', 'console'],
            'level': 'INFO',
            'propagate': True,
        }
    }
}

```

+ 보안구룹 RDS 인바운드 추가해주기(admin으로 접속했을때 접속안됨 그래서 추가)
	+ SecurityGroup for ElasticBeanstalk environment 그룹ID를 소스에 입력


### 실시간으로 디버그 보기

+ `docker exec cantainer_name tail -f /srv/project/.log/error.log`

### 디버그 자동화 만들기

+ [eb ssh문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/eb3-ssh.html)
+ `--command`사용하기 test
	+ `eb ssh --comaand 'pwd'`
	+ `eb ssh --command 'sudo docker ps'`
	+ `eb ssh --command 'sudo docker ps -q'`
	+ `eb ssh --command 'sudo docker exec $(sudo docker ps -q) tail -f /srv/project/.log/error.log /srv/project/.log/info.log'`

+ 자동화 만들기

```
> root/log.sh

#!/usr/bin/env bash
LOG_ERROR='/srv/project/.log/error.log'
LOG_INFO='/srv/project/.log/info.log'
CMD='sudo docker exec $(sudo docker ps -q) tail -f'
eb ssh --command "$CMD $LOG_ERROR $LOG_INFO"

```

### ELB Health check 4xx에러해결하기

+ Add EC2 Private IP to ALLOWED_HOSTS
	+ Django애플리케이션이 동작하는 각 EC2인스턴스에서는 자신의 PrivateIP를 ALLOWED_HOSTS에 추가해야 한다.

```
# settings/production.py

def is_ec2_linux():
    """
    Detect if we are running on an EC2 Linux Instance
    See http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/identify_ec2_instances.html
    """
    if os.path.isfile("/sys/hypervisor/uuid"):
        with open("/sys/hypervisor/uuid") as f:
            uuid = f.read()
            return uuid.startswith("ec2")
    return False


def get_linux_ec2_private_ip():
    """Get the private IP Address of the machine if running on an EC2 linux server"""
    from urllib.request import urlopen
    if not is_ec2_linux():
        return None
    try:
        response = urlopen('http://169.254.169.254/latest/meta-data/local-ipv4')
        ec2_ip = response.read().decode('utf-8')
        if response:
            response.close()
        return ec2_ip
    except Exception as e:
        print(e)
        return None
        
private_ip = get_linux_ec2_private_ip()
if private_ip:
    ALLOWED_HOSTS.append(private_ip)

```

### django commands

+ [django-admin commands 문서](https://docs.djangoproject.com/en/2.2/howto/custom-management-commands/)

```
# management, commands 패키지폴더로 만들기
> app/members/management/commands/create_dummy_user.py


from django.contrib.auth import get_user_model
from django.core.management import BaseCommand
from django.utils.crypto import get_random_string

User = get_user_model()


class Command(BaseCommand):
    def handle(self, *args, **options):
        username = f'dummy_{get_random_string(length=10)}'
        User.objects.create_user(username=username)
        self.stdout.write(f'User {username} created')

```


### EB hooks

+ Elastic Beanstalk은 표준화 된 디렉토리 구조를 후크에 사용
+ [플랫폼 후크 문서](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/custom-platform-hooks.html)

```
> 터미널에서 eb ssh로 접속
> cd /opt/elasticbeanstalk/hooks/appdeploy/post/ 경로접속

> vi app_01_create_dummy_user.sh 안에 테스트 코딩 넣기
#!/usr/bin/env bash
sudo docker exec $(sudo docker ps -q) python3 /srv/project/app/manage.py create_dummy_user
# 실행시 디비에 User가 추가되는지 확인

```


### ebextensions

+ 구성 파일 (.ebextensions)을 사용하여 고급 환경 사용자 지정
	+ [ebextensions문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/ebextensions.html)
	
+ hooks/appdeploy/post/ 안쪽에 있는 스크립트들은 배포가 완료된후에 실행된다
+ hooks안에는 우리가 실행할 스크립트들을 어떤 시점에 맞게 넣어놔서 배포과정을 세밀하게 조정할수 있다
+ 앞번호순서대로 실행된다(00,01순으로)

```
# hooks에 test한 스트립트가 남아있는것을 지우는 내용
> root/.ebextensions/00_commands.config

commands:
  01_remove_sript:
    command: "rm /opt/elasticbeanstalk/hooks/appdeploy/post/app_*"


# db migrate해주는 내용
> root/.ebextensions/01_files.config

files:
  "opt/elasticbeanstalk/hooks/appdeploy/post/app_01_migrate.sh":
    mode: "000755"
    owner: root
    group: root
    content: |
      #!/usr/bin/env bash
      sudo docker exec $(sudo docker ps -q) python3 /srv/project/app/manage.py migrate
      
```


## Route53 설정

+ 도메인 등록

```
> Route53 설정

DNS 관리 (시작하기 클릭) -> 호스팅 영역 생성 -> 도메인 이름넣어주고, 생성 클릭

-> 도메인싸이트와서 사용하는 도메인 체크하고 네임서버 주소변경하기 체크하고 신청누르기

-> Route53 네임서버 값을 도메인 네임서버에 넣어주고 변경하기 (TTL은 1m 체크하고 레코드 세트 저장 누르기)

-> 레코드 세트 생성 클릭 -> 이름(x, www, api)3개 만들기, 유형 A - IPv4 주소 선택

-> Alias에 YES 체크, 대상 Load Balancer선택후  생성 클릭

```

```
> .config/app.nginx

server_name 도메인이름 추가해주기

> config/settings/production.py

ALLOWED_HOSTS에도 도메인이름 추가해주기

```

## Sentry 설정

+ 에러가 날때마다 메일로 알려줌

```
로그인후 -> Proejcts 클릭, Popular(Django 선택, proejct_name, team 적고) create 클릭

-> sentry패키지 설지 후 settings/production.py안에 SDK구성하기 (sentry사이트에 하는법 나와있음)

```

+ DSN은 보일필요가 없어서 secrets에 넣어서 관리하기


## ACM

+ 도메인 이름과 도메인 내의 여러 이름을 보호할수 있다.(인증서)

```
> 인증서 프로비저닝

인증서 프로비저닝 시작하기 클릭 -> 공인 인증서 요청 클릭 -> 도메인 이름 추가 후 다음 클릭

-> DNS 검증 검토클릭 -> 확인 및 요청 클릭 -> Route53에 레코드 생성 후 계속 클릭 

-> 검증상태 성공 확인후 -> EC2의 로드밸런서의 리스너 추가하기(HTTPS선택, 작업추가(다음으로 전달), ACM에서 권장) 저장 클릭

-> 보안 그룹의 Elastic Beanstalk created의 인바운드로가서 편집(https 선택후 저장)

```

