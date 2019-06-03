---
layout: post
title: "[Django] Django Project Settings"
categories: posts
tags: django
---

# Django Proejct settings

## 초기설정

```
project폴더 생성 -> git,gitignore설정 

-> pyenv설정(pyenv virtualenv 3.6.8 가상환경사용할이름) 

-> pyenv local 설정(pyenv local 가상환경사용할이름) 

-> django 설치(pip install django) 

-> project만들기(django-admin startproject config) 

-> project명 app으로 바꾸기(터미널에서 mv config app) 

```

### Secrets

`.secrets/base.json`

```json
{
  "SECRET_KET" : "<Django secret key>"
}
```

### ROOT_DIR

```config/settings.py
# settings.py

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
ROOT_DIR = os.path.dirname(BASE_DIR)
STATIC_DIR = os.path.join(BASE_DIR, 'static')
TEMPLATES_DIR = os.path.join(BASE_DIR, 'templates')
SECRETS_DIR = os.path.join(ROOT_DIR, '.secrets')
secrets = json.load(open(os.path.join(SECRETS_DIR, 'base.json')))
SECRET_KEY = secrets['SECRET_KEY']
DEBUG = True

# Static
STATICFILES_DIRS = [
    STATIC_DIR,
]
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
MEDIA_ROOT=os.path.join(ROOT_DIR, '.media')
```

## User 구현

+ User 구현하고 project 시작하기
+ members app 추가(User구현)

```members
# models.py

from django.contrib.auth.models import AbstractUser


class User(AbstractUser):
    pass

```

```config/settings.py
# settings.py 안에 추가

# Auth
AUTH_USER_MODEL = 'members.User'

```

