---
layout: post
title: "[Djangogirls-tutorial] 장고걸스 튜토리얼 사용해보기"
categories: posts
tags: djangogirls
---

# [Djangogirls-tutorial] 장고걸스 튜토리얼 사용해보기


## 튜토리얼 하기전 setting
```
djangogirls-tutorial 폴더 생성 -> pyenv 설정(가상환경 설정) 
-> git, .gitignore( git, linux, macos, python, pycharm+all, jupyternotebook, django)설정 
-> pip install django~=1.11.0(설치) 
-> django-admin.py startproject mysite

Pycharm Interpreter 설정
macOS
	/usr/local/var/pyenv/versions/
```

## 폴더 설명

```
djangogirls-tutorial/ 	<- 프로젝트 폴더
	app/		<- Django 폴더, 
		config/		<- Django 설정 패키지
			settings.py	<- Django 설정 모듈
		manage.py		<- Django 유틸리티 모듈
```

## 가상환경 웹서버 연결 
```
app폴더 안에 manage.py 파일 위치에서 실행하기
> python manage.py runserver

브라우저 창
127.0.0.1:8000
```

### models

+ 데이터베이스를 편하기 보기위해서 설치
	+ `brew cask install db-browser-for-sqlite`

```
class Post(models.Model):
    author = models.ForeignKey('auth.User', on_delete=models.CASCADE, )
    title = models.CharField(max_length=200)
    text = models.TextField(blank=True)
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
        
--------	models.py에 코딩후 --------->

config/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'blog',		<-----------추가
]
```
+ 데이터베이스 추가
	+ `python manage.py makemigrations blog`
+ 데이터베이스 적용
	+ `python manage.py migrate`



### url, view 연결

```
# admin.py

from django.contrib import admin
from .models import Post

admin.site.register(Post)

```

+ 유저만들기
	+ `python manage.py createsuperuser`
+ 관리자 모드 
	+ 127.0.0.1:8000/admin


```
# config/urls.py

from blog import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.post_list),        <---추가
]
```
```
# blog/views.py

from django.http import HttpResponse


def post_list(request):
    # 현재 지역에 맞는 날짜&시간 객체 할당
    current_time = timezone.now()

    return HttpResponse(
        '<html>'
        '<body>'
        '<h1>Post List</h1>'
        '<p>{}</p>'
        '</body>'
        '</html>'.format(
            # 날짜&시간 객체가 가진 정보를 문자열로 변환
           current_time.strftime('%Y. %m. %d<br>%H:%M::%S')
        )
    )
   
---- 브라우저 확인 하면 post_list함수 리턴된 결과 표시 -----

def post_list(request):
    context = {
        'pokemon': random.choice(['피카츄', '라이츄', '파이리']),
    }
    return render(
        request=request,
        template_name='blog/post_list.html',
        context=context,
    )
    # return render(request, 'blog/post_list.html', context)

----------- render 방법 두가지 ---------------
```

```
MVC모델		       Django

Model	    <--->	Model (models.py)
View	    <--->	Template
Controller  <--->	View (views.py)
----------------------------------------------------

runserver (웹 서버)
Django application
	urlresolver (config.urls)
	
	
127.0.0.1:8000 -> runserver -> urlresolver 
	-> ''와 매치
	-> 해당 요청을 blog.views.post_list에 보냄
	-> 함수에서는 받은 요청을 처리한 결과를 리턴
	-> 리턴된 결과는 브라우저에 표시
```


### os.path 모듈 (경로를 만들어주는 역활)

```
import os

current_path = os.path.abspath(__file__)    <-- 실행중인 파일의 경로
parent_dir = os.path.dirname(current_path)  <-- 상위 폴더
blog_dir = os.path.join(parent_dir, 'blog') <-- 하위 폴더
blog_urls = os.path.join(blog_dir, 'urls.py') <-- 하위 폴더
```

### ORM 쿼리셋

+ 모든 객체 조회하기
	+ 모델 클래스명.objects.all()
+ 객체 생성하기
	+ 모델 클래스명.objects.create(author='',title='' ....)
+ 객체 가져오기
	+ 모델 클레스명.objects.get(username='')
+ 필터링
	+ 모델 클레스명.objects.filter(title__contains='')
+ 정렬하기
	+ 모델 클레스명.objects.order_by('title') <- 오름차순
	+ 모델 클레스명.objects.order_by('-title') <- 내림차순

## template, view

```
# blog/views.py( Post 클레스 created_date를 내림차순으로 리턴 )

def post_list(request):
	posts = Post.objects.order_by('-created_date')
	context = {
		'posts': posts,
   	}
   return render(request, 'blog/post_list.html', context)
```

```
# templates/blog/post_list.html ( 템플릿으로 Post.title 출력 결과 보기 )

<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1>Post List</h1>
    <ul>
    {% for post in posts %}
        <li>{{ post.title }}</li>
    {% endfor %}
    </ul>
</body>
</html>
```


## bootstrap 사용하기

```
# config/settings.py

STATIC_DIR = os.path.join(BASE_DIR, 'static')

STATICFILES_DIRS = [
	STATIC_DIR,
]

STATIC 경로 만들어주기 (경로를 만들어줘야 HTML에서 bootstrap 불러올수있음)

```


```
# static load 방법 (html 파일안에 추가해주기)

{% load static %}	<------ 맨위
<link rel="stylesheet" href="{% static 'bootstrap/css/bootstrap.css' %}">  
```

## templates 용어

```
linebreaksbr      	<- 엔터가 들어가있는 데이터 줄바꿈해주는 명령어 
truncatechars:200 	<- 200글자 이후에는 짤림
{{ test }} <-- 변수를 의미
{% extends 'base.html' %} <- base.html참조
{% block content %}{% endblock %} <- html파일 나눌때 사용
{% url 'urlname넣기' pk=post.pk %} <- href url경로 쓸때 사용

```

## 폼사용하기

```
{% extends 'base.html' %}

{% block content %}
<h2>새 포스트 작</h2>
<form action="" method="POST">
    {% csrf_token %}
    <div class="form-group">
        <label for="id-title">제목</label>
        <input id="id-title" type="text" name="title" class="form-control">
    </div>
    <div class="form-group">
        <label for="id-text">내용</label>
        <textarea name="text" id="id-text" cols="30" rows="10"class="form-control"></textarea>
    </div>
    <button type="submit" class="btn btn-primary btn-block">작성</button>
</form>
{% endblock %}

# method=POST 방식일때는 csrf 필요
```


## views에 데이터베이스 추가, 업데이트 하는 코딩

```
# db에 데이터 생성 코드

def post_create(request):
    if request.method == 'POST':
        title = request.POST['title']
        text = request.POST['text']
        post = Post.objects.create(
            author=request.user,
            title=title,
            text=text,
        )
        # 글 목록 페이지로 Redirect응답을 보냄
        # next_path = reverse('post-list')
        # return HttpResponseRedirect(next_path)
        
        # URL Name으로부터의 reverse과정이 추상화되어있음
        return redirect('post-list')
    else:
        return render(request, 'blog/post_create.html')
```

```
# 데이터 수정 코드..

def post_update(request, pk):
    post = Post.objects.get(pk=pk)
    # pk에 해당하는 Post Instance를 'post'키 값으로 템플릿 전달
    if request.method == 'POST':
        # form으로부터 전달된 데이터를 변수에 할당
        title = request.POST['title']
        text = request.POST['text']

        # 수정할 Post Instance의 속성에
        # 전달받은 데이터의 값을 할당
        post.title = title
        post.text = text

        # DB에 변경사항을 Update
        post.save()
        
        # post-detail로 이동, reverse()에 pk를 사용
        return redirect('post-detail', pk=pk)
    else:

        context = {
            'post': post
        }
        return render(request, 'blog/post_update.html', context)

```


## Paginator

[Paginator문서](https://docs.djangoproject.com/en/2.1/topics/pagination/)

```
# view

def post_list(request):
    # 생성일자 내림차순으로 정렬된 모든 POST 목록을
    # 5개씩 나눔 Paginator 객체 생성
    paginator = Paginator(
        Post.objects.order_by('-created_date'), 5)
   
    page = request.GET.get('page')

    try:
        # page변수가 가진 값에 해당하는 page를
        # Paginator에서 가져오기 위해 시도
        posts = paginator.page(page)
    except PageNotAnInteger:
        # page변수가 정수가 아니어서 발생한 예외의 경우
        # -> 무조건 첫 페이지를 가져옴
        posts = paginator.page(1)
    except EmptyPage:
        # page변수에 해당하는 Page에 내용이 없는 경우
        # (3페이지 까지만 가능한데 5페이지 호출 등)
        # -> 무조건 마지막 페이지를 가져옴
        posts = paginator.page(paginator.num_pages)


    context = {
        'posts': posts,
    }
    return render(request, 'blog/post_list.html', context)
```

```
# template

<nav>
        <ul class="pagination">
            {% if posts.has_previous %}
            <li class="page-item">
                <a href="?page={{ posts.previous_page_number }}" class="page-link">이전</a>
            </li>
            {% endif %}

            <li class="page-item active">
                <a href="" class="page-link">
                    {{ posts.number }} of {{ posts.paginator.num_pages }}
                    <span class="sr-only">{current}}</span>
                </a>
            </li>

            {% if posts.has_next %}
            <li class="page-item">
                <a href="?page={{ posts.next_page_number }}" class="page-link">다음</a>
            </li>
            {% endif %}
        </ul>
</nav>
```






