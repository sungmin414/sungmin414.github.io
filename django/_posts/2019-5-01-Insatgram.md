---
layout: post
title: "[Django-instagram] 인스타그램 만들어보기"
categories: posts
tags: django
---


# Instagram 만들어보기


## 초기 설정

```
instagram폴더 생성 -> git,gitignore설정 
-> pyenv설정(pyenv virtualenv 3.6.8 가상환경사용할이름) 
-> pyenv local 설정(pyenv local 가상환경사용할이름) 
-> django 설치(pip install django) 
-> project만들기(django-admin startproject mysite) 
-> project명 app으로 바꾸기(터미널에서 mv mysite app) 
-> mysite 설정파일 config로 이름바꾸기(charm에서 바꾸기 체크 2개다하기) 
```

### pip 기록하기 

+ pip freeze
	+ 패키지 보기
+ pip freeze > requirements.txt
	+ 패키지 기록



## 모델링

인스타그램 만들기

## Installation

### Requirements

#### python, packages

- Python(3.6)

```
pip install -r requirements.txt
```

#### Secrets

`.secrests/base.json`

```json
{
  "SECRET_KEY": "<Django SECRET_KEY>",
  "FACEBOOK_APP_ID": <FACEBOOK_ID>,
  "FACEBOOK_APP_SECRET": <Facebook APP Secret>"
}

```

### models

+ 포스트(Post)
	+ 작성자 (author)
	+ 사진 (photo)
	+ 댓글 목록

+ 포스트 댓글 (comment)
	+ 해당 포스트(post)
	+ 작성자 (author)
	+ 내용 (content)
	+ 해시태그
	+ 멘션
+ 사용자 (User)
	+ 사용자명 (username)
	+ 프로필 이미지 (img_profile)
	+ 이름 (name)
	+ 웹사이트 (site)
	+ 소개 (introduce)



### `app_user, app_posts` 모델링
```
# members/models.py

from django.db import models


from django.db import models


class User(models.Model):
    username = models.CharField(
        '아이디',
        max_length=50,
        # 중복검사
        unique=True,
    )
    img_profile = models.ImageField(
        '프로필 이미지',
        upload_to='user',
        blank=True,
    )
    name = models.CharField('이름', max_length=30, blank=True)
    site = models.URLField('사이트', max_length=150, blank=True)
    introduce = models.TextField('소개', blank=True)

    def __str__(self):
        return self.username

    class Meta:
        verbose_name = '사용자'
        verbose_name_plural = f'{verbose_name} 목록'

``` 

> posts app 모델에서 members app의 User모델 클래스를 ForeignKey로 참조시  `<AppName>.<modelName>`으로 참조하면된다.

```
posts/models.py


from django.db import models


class Post(models.Model):
    author = models.ForeignKey(
        # <AppName>.<modelName>
        # 'members.User',
        'members.User',
        on_delete=models.CASCADE,
        verbose_name='작성자',
    )
    photo = models.ImageField('사진', upload_to='post')

    class Meta:
        verbose_name = '포스트'
        verbose_name_plural = f'{verbose_name} 목록'


class Comment(models.Model):
    post = models.ForeignKey(
        'Post',
        on_delete=models.CASCADE,
        verbose_name='포스트',
    )
    author = models.ForeignKey(
        'members.User',
        on_delete=models.CASCADE,
        verbose_name='작성자',
    )
    content = models.TextField('댓글 내용')
    tags = models.ManyToManyField(
        'HashTag',
        blank=True,
        verbose_name='해시태그 목록',
    )

    class Meta:
        verbose_name = '댓글'
        verbose_name_plural = f'{verbose_name} 목록'


class HashTag(models.Model):
    name = models.CharField('태그명', max_length=100)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = '해시태그'
        verbose_name_plural = f'{verbose_name} 목록'




```

+ 관리자 페이지에서 만든 모델들이 정상 작동하는가 확인하고 view로 옮기는게 좋음
+ 각 app에 admin 등록해서 관리자 페이지 들어가보기

```
# members/admin.py

from django.contrib import admin

from .models import User

admin.site.register(User)




# posts/admin.py

from django.contrib import admin
from .models import Post, Comment, HashTag

admin.site.register(Post)
admin.site.register(Comment)
admin.site.register(HashTag)
```

### settings

+ 경로 지정

```
import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# 사용자가 업로드한 파일이 저장될 Base디렉토리 (settings.MEDIA_ROOT)

ROOT_DIR = os.path.dirname(BASE_DIR)
MEDIA_ROOT = os.path.join(ROOT_DIR, '.media')
TEMPLATES_DIR = os.path.join(BASE_DIR, 'templates')
STATIC_DIR = os.path.join(BASE_DIR, 'static')

# 유저가 업로드한 파일에 접근하고자 할 때의 prefix URL (settings.MEDIA_URL)
# FileField, MediaField의 URL의 아래 설정 기준으로 바뀜
MEDIA_URL = '/media/'
STATIC_URL = '/static/'

# 정적 파일을 검색할 경로 목록
STATICFILES_DIRS = [
    STATIC_DIR,
]

```

### urls

+ MEDIA_URL로 시작하는 URL은 static() 내의 serve() 함수를 통해
+ MEDIA_ROOT기준으로 파일을 검색함

```
# static을 추가해주므로서 관리자모드에서 사진을올리고 올린파일을누르면 사진을 볼수있음

from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import path


urlpatterns = [
    path('admin/', admin.site.urls),

]
# MEDIA_URL로 시작하는 URL은 static() 내의 serve() 함수를 통해
# MEDIA_ROOT기준으로 파일을 검색함
urlpatterns += static(
    prefix=settings.MEDIA_URL,
    document_root=settings.MEDIA_ROOT,
)

```

### bootstrap 사용하기

```
# 부트스트랩, nomalize 파일 생성 및 경로

app/static 폴더생성
	# 부트스트랩 경로
	-> bootstrap/css/bootstrap.css
	# nomalize 경로
	-> css/normalize.css
```

```
# base.html에서 static load하기

{% raw %}
{% load static %}		<- html파일 안에서 맨위에 작성

# html head안에 static/css link하기

<!-- Normalize.css: 브라우저마다 다른 css기본 설정값들을 같게 해줌 -->
<link rel="stylesheet" href="{% static 'css/normalize.css' %}">

<!-- Bootstrap CSS -->
<link rel="stylesheet" href="{% static 'bootstrap/css/bootstrap.css' %}">
{% endraw %}
```


### forms

```
# post_create.html

<div class="form-group">
            <label for="id-photo">사진첨부</label>
            <input type="file" id="id-photo" name="photo" class="form-control-file">
</div>


# posts/forms.py

from django import forms


class PostCreateForm(forms.Form):
    photo = forms.ImageField()


```

+ `post_create.html` 코딩한 부분을 forms.py의 form클래스를 정의하면
photo에 ImageField를 정의했는데 form클래스에서 post_create.html정의 해논 label과 input을 자동으로 만들어서 할당해줌(form클래스를 template에서 부를때는 {{ form }}으로 불러옴

+ form클래스에 css사용하기( widget 속성사용 )

```
예)

from django import forms


class PostCreateForm(forms.Form):
    photo = forms.ImageField(
        widget=forms.FileInput(
            # HTML위젯의 속성 설정
            # form-control-file클래스를 사
            attrs={
                'class': 'form-control-file',
            }
        )
    )
    comment = forms.CharField(
        # 반드시 채워져 있을 필요는 없음
        required=False,
        # HTML렌더링 위젯으로 textarea를 사용
        widget=forms.Textarea(
            attrs={
                'class': 'form-control',
            }
        )
    )
```


### `AUTH_USER_MODEL`

+ django가 기본적으로 제공하는 User클래스
+ models.py에서 사용할때
	+ from django.conf import settings
	+ `settings.AUTH_USER_MODEL` 




### Login, logout, signup구현

```
# forms.py

from django import forms


class LoginForm(forms.Form):
    username = forms.CharField(
        # 일반 input[type=text]
        # form-control CSS클래스 사용
        # (Bootstrap규칙)
        widget=forms.TextInput(
            attrs={
                'class': 'form-control'
            }
        )
    )
    password = forms.CharField(
        # input[type=password]
        widget=forms.PasswordInput(
            attrs={
                'class': 'form-control'
            }
        )
    )


class SignupForm(forms.Form):
    username = forms.CharField(
        widget=forms.TextInput(
            attrs={
                'class': 'form-control',
            }
        )
    )
    password1 = forms.CharField(
        widget=forms.PasswordInput(
            attrs={
                'class': 'form-control',
            }
        )
    )
    password2 = forms.CharField(
        widget=forms.PasswordInput(
            attrs={
                'class': 'form-control',
            }
        )
    )
```

```
# views.py

from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.models import User
from django.http import HttpResponse
from django.shortcuts import render, redirect

from .forms import LoginForm, SignupForm


def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(username=username, password=password)
        if user is not None:
            # 인증성공시
            login(request, user)
            return redirect('posts:post-list')
        else:
            form = LoginForm(),
            context = {
                'form': form,
                'error': '정보가 일치하지 않습니다'
            }
            return render(request, 'members/login.html', context)

    else:
        form = LoginForm()
        context = {
            'form': form,
        }
        return render(request, 'members/login.html', context)


def logout_view(request):
    if request.method == 'POST':
        logout(request)
        return redirect('posts:post-list')


def signup_view(request):
    context = {
        'form': SignupForm(),
    }
    if request.method == 'POST':
        username = request.POST['username']
        password1 = request.POST['password1']
        password2 = request.POST['password2']

        if User.objects.filter(username=username).exists():
            context['error'] = f'({username})은 이미 사용중입니다'
        elif password1 != password2:
            context['error'] = '비밀번호와 비밀번호 확인란의 값이 일치하지 않습니다'
        else:
            # create_user메서드는 create와 달리 자동으로 password해싱을 해줌
            user = User.objects.create_user(
                username=username,
                password=password1,
            )
            login(request, user)
            return redirect('posts:post-list')
    return render(request, 'members/signup.html', context)

```

```
# login.html

{% raw %}
{% extends 'base.html' %}

{% block content %}
{#총 12칸#}
{#col-lg: 큰화면#}
{#    4칸을 차지, 4칸부터 시작#}
{#col-md: 중간 화면#}
{#    6칸을 차지, 3칸부터 시작#}
{#col-sm: 작은 화면#}
{#    8칸을 차지, 2칸부터 시작#}
<div class="col
            col-lg-4 offset-lg-4
            col-md-6 offset-lg-3
            col-sm-8 offset-sm-2">
    <h2 class="text-center">로그인</h2>
    <form action="" method="POST">
        {% csrf_token %}
        {% for field in form %}
        <div class="form-group">
            <label for="{{ field.id_for_label }}">
                {{ field.label }}
            </label>
            {{ field }}
        </div>
        {% endfor %}
    <button type="submit" class="btn btn-lg btn-primary btn-block">로그인</button>
    </form>
</div>
{% endblock %}
{% endraw %}
```

### RedirectView 제네릭뷰

+ `path('', RedirectView.as_view(pattern_name='posts:post-list'), name='index')`
	+ views.py을 사용하지않고 설정 url에서 원하는 url로 바로 연결시켜주는 뷰


### 데이터 유효성 검증을 제외한 Form사용

```
# views.py

def signup_view(request):
    context = {}
    if request.method == 'POST':
        # POST로 전달된 데이터를 확인
        # 올바르다면 User를 생성하고 post-list화면으로 이동
        # (is_valid()가 True면 올바르다고 가정)
        form = SignupForm(request.POST)
        if form.is_valid():
            user = User.objects.create_user(
                username=form.cleaned_data['username'],
                password=form.cleaned_data['password1'],
            )
            login(request, user)
            # form이 유효하면 여기서 함수 실행 종료
            return redirect('posts:post-list')
        # form이 유효하지 않을 경우, 데이터가 바인딩된 상태로 if-else구문 아래의 render까지 이동
    else:
        # GET요청시 빈 Form을 생성
        form = SignupForm()
    # GET 요청시 또는 POST로 전달된 데이터가 올바르지 않을 경우
    # signup.html에 빈 Form또는 올바르지 않은 데이터에 대한 정보가 포함된 Form을 전달해서
    # 동적으 form을 렌더링
    context['form'] = form
    return render(request, 'members/signup.html', context)
```

### SignupForm의 Validation구현

```
# forms.py

    def clean_username(self):
        # username이 유일한지 검사
        data = self.cleaned_data['username']
        if User.objects.filter(username=data).exists():
            raise forms.ValidationError('이미 사용중 사용자명입니다')

    def clean_password2(self):
        password1 = self.cleaned_data['password1']
        password2 = self.cleaned_data['password2']
        if password1 != password2:
            raise forms.ValidationError('비밀번호와 비밀번호 확인란의 값이 다릅니다')
        return password2
```

### Form및 Field가 가진 Errors를 템플릿에서 출력

```
# template

{% raw %}
{#                Form자체의 Error들을 출력#}
            {% if form.non_field_errors %}
                {% for error in form.non_field_errors %}
                    <small class="form-text text-muted">{{ error }}</small>
                {% endfor %}
            {% endif %}

{#            Form이 가진 field들을 순회#}
            {% for field in form %}
                <div class="form-group">
                    <label for="{{ field.id_for_label }}">
                        {{ field.label }}
                    </label>
                    {{ field }}

{#                    각 Field가 가진 Error들을 input밑에 출력#}
                    {% for error in field.errors %}
                        <small class="form-text text-muted">{{ error }}</small>
                    {% endfor %}
{% endraw %}            
```

### login_required

```
# settings.py

LOGIN_URL = 'members:login'
```

```
# views.py

# GET parameter에 'next'가 전달되었다면
            #  해당 키의 값으로 redirect
            next_path = request.GET.get('next')
            if next_path:
                return redirect(next_path)
```



### `tag_post_list, tag_search`

```
# views.py

def tag_post_list(request, tag_name):
    # Post중, 자신에게 속한 Comment가 가진 HashTag목록 중
    # Post목록을 posts변수에 할당
    # context에 담아서 리턴 render
    # HTML: /posts/tag_post_list.html
    posts = Post.objects.filter(comments__tags__name=tag_name)
    context = {
        'posts': posts
    }
    return render(request, 'posts/tag_post_list.html', context)


def tag_search(request):
    # request.GET으로 전달된
    # search_keyword값을 적절히 활용해서
    # 위의 tag_post_list view로 redirect
    # URL: 'posts/tag-search/'
    # URL Name : 'posts:tag-search'
    # Template: 없
    search_keyword = request.GET.get('search_keyword')
    substituted_keyword = re.sub(r'#|\s+', '', search_keyword)
    return redirect('tag-post-list', substituted_keyword)

```

### User모델 참조
+ `get_user_model() 함수 사용

```
# members/forms.py

User = get_user_model() 
```
+ img_profile 비어있을경우 property사용해서 예외처리

```
# members/models.py

    @property
    def img_profile_url(self):
        # 자신의 img_profile필드에 내용이 있다면
        # 해당 파일로 접근할 수 잇는 url을 리턴
        if self.img_profile:
            return self.img_profile.url
        # 없다면 static폴더에서 뒷 결로 파일을 url을 리턴
        return static('images/blank_user.png')
```


### UserProfileForm 사용해 User모델 기능 수정 추가

```
# members/forms.py

class UserProfileForm(forms.ModelForm):
    class Meta:
        model = User
        fields = [
            'email',
            'last_name',
            'first_name',
            'img_profile',
            'site',
            'introduce',
        ]

```

### post 좋아요 구현

```
# members/models.py

   def like_post_toggle(self, post):
        # 자신에게 연결된 PostLike중, post값이 매개변수의 post인 PostLike가 있다면 가져오고, 없으면 생성
        postlike, postlike_created = self.postlike_set.get_or_create(post=post)
        # 생성되었다면 없다가 생겼다는 말이므로 (새로 종아요를 누름) 따로 처리 필요없음
        # 생성되지 않았다면 이미 있었다는 말이므로 toggle처리를 위해 삭제
        if not postlike_created:
            postlike.delete()

```

```
# posts/models.py

class Post(models.Model):
    author = models.ForeignKey(
        # <AppName>.<modelName>
        # 'members.User',
        # User,
        # 'auth.User'
        # Django가 기본적으로 제공하는 User클래스
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        verbose_name='작성자',
    )
    # auto_now_add: 객체가 처음 생성될때의 시간 저장
    # auto_now: 객체의 save()가 호출될 때 마다 시간 저장
    photo = models.ImageField('사진', upload_to='post')
    created_at = models.DateTimeField(auto_now_add=True)
    modified_at = models.DateTimeField(auto_now=True)

    like_users = models.ManyToManyField(
        settings.AUTH_USER_MODEL,
        through='PostLike',
        related_name='like_posts',
        related_query_name='like_post',
    )

    class Meta:
        verbose_name = '포스트'
        verbose_name_plural = f'{verbose_name} 목록'
        ordering = ['-pk']

    def like_toggle(self, user):
        # 전달받은 user가 이 Post를 Like한다면 해제
        # 안되어있다면 Like처리
        postlike, postlike_created = self.postlike_set.get_or_create(user=user)
        if not postlike_created:
            postlike.delete()
```

```
# posts/views.py

def post_like_toggle(request, post_pk):
    if request.method == 'POST':
        post = get_object_or_404(Post, pk=post_pk)
        post.like_toggle(request.user)
        # post_list의 좋아요 눌렀을때 위치 지정
        url = reverse('posts:post-list')
        return redirect(url + f'#post-{post_pk}')

```

```
# post_list.html
{% raw %}
{% extends 'base.html' %}

{% block content %}
    <div>
        {% for post in posts %}
           <!-- 이 div가 lg(width >= 992px)일때, 4/12의 크기를 기준으로 시작함 -->
            <div id="post-{{ post.pk }}" class="col col-lg-4 offset-lg-4 mb-4">
                <!-- card모양에 대해 미리 정의된 클래스 -->
                <div class="card">
                    <!-- 작성자 정보를 나타낼 header부분 -->
                    <div class="card-header">
                        <div style="width:30px; height:30px; display: inline-block; vertical-align: middle;">
                            <a href="" style="background-image: url('{{ post.author.img_profile_url }}');
                                    display: inline-block; width: 100%; height: 100%; background-size: cover;
                                    background-position: center center; vertical-align: middle;"
                                    class="rounded-circle"></a>
                        </div>
                        <span>{{ post.author }}</span>
                    </div>
                    <!-- Card의 본문 부분 -->
                    <div class="card-body">
                        <img src="{{ post.photo.url }}" class="card-img-top">
                        {% if user.is_authenticated %}
                        <form action="{% url 'posts:post-like-toggle' post_pk=post.pk %}" method="POST">
                            {% csrf_token %}
                            <button type="submit" class="btn btn-primary">
                                {% if user in post.like_users.all %}
                                    좋아요 해제
                                {% else %}
                                    좋아요
                                {% endif %}
                            </button>
                        </form>
                        {% endif %}
                        <div>
                            <span>좋아요 하는사람</span>
                            <strong>{{ post.like_users.all|join:", " }}</strong>
                        </div>
                        <ul class="list-unstyled">
                            {% for comment in post.comments.all %}
                                <li>
                                    <strong>{{ comment.author }}</strong>
                                    <span>{{ comment.html|safe }}</span>
                                </li>
                            {% endfor %}
                        </ul>
{#                        댓글 작성 form구현#}
{#                        1. 유저가 로그인 한 경우에만 보여지도록 함#}
{#                        2. 내부요소는 atextarea[name=content]와 버튼하나#}
{#                        3. action속성의 값을 위의 'comment_create' view에 해당하는 URL로 지정#}
                        {% if user.is_authenticated %}
                            <form action="{% url 'posts:comment-create' post_pk=post.pk %}" method="POST">
                            {% csrf_token %}
                                {{ comment_form }}
                                <button type="submit" class="btn btn-primary btn-block">작성</button>
                            </form>
                        {% endif %}
                    </div>
                </div>
            </div>
        {% endfor %}
    </div>
{% endblock %}
{% endraw %}
```


### OAuth, FaceBook Login

[OAuth란?](https://ko.wikipedia.org/wiki/OAuth)


## JavaScript | Ajax

+ Ajax는 JavaScript의 라이브러리중 하나이며 Asynchronous Javascript And Xml(비동기식 자바스크립트와 xml)의 약자이다.
+ 브라우저가 가지고있는 XMLHttpRequest 객체를 이용해서 전체 페이지를 새로 고치지 않고도 페이지의 일부만을 위한 데이터를 로드하는 기법 이며 Ajax를 한마디로 정의하자면 JavaScript를 사용한 비동기 통신, 클라이언트와 서버간에 XML 데이터를 주고받는 기술이라고 할 수 있겠습니다.

```
> posts/apis.py

import json

from django.http import HttpResponse

from .models import HashTag


def tag_search(request):
    keyword = request.GET.get('keyword')
    tags = []
    if keyword:
        tags = list(HashTag.objects.filter(name__istartswith=keyword).values())
    result = json.dumps(tags)
    return HttpResponse(result, content_type='application/json')

```

```
> templates/base.html

<script>
    // 검색창 밑의 결과 창 요소
    var searchList = $('ul.search-list');

    // 검색창에서 이벤트(keyup)가 일어나면 뒤의 function을 실행함
    $('#search-input').keyup(function (e) {
        // 검색창에 입력된 값을 위 이벤트가 실행된 순간 가져옴
        var content = $('#search-input').val();

        // 만약 검색창에 있는 값의 길이가 0이면 결과창을 숨김
        if (content.length == 0) {
            searchList.hide();
        } else {
            searchList.show();
        }

        // API에 요청
        // /posts/api/tag-search/에 GET요청
        // data에 keyword로 검색창에 입력되었던 값을 보냄
        // -> GET Parameter로 알아서 번역
        $.ajax({
            method: 'GET',
            url: 'http://localhost:8000/posts/api/tag-search/',
            data: {
                keyword: content
            }
        })
        // 요청이 성공했을 경우
        .done(function (response) {
            // response에 Tag 객체의 리스트가 전달됨
            var tags = response;
            console.log(tags);
            // 검색 결과창을 비우고
            searchList.empty();
            // Tag 객체 리스트를 순회
            for (var i = 0; i < response.length; i++) {
                // 현재 순회중인 객체를 index(i)를 사용해 직접 꺼냄
                var curTag = response[i];
                // 결과창에 직접 HTML요소를 삽입
                searchList.append('<li><a href="/explore/tags/' + curTag.name +'/">' + curTag.name + '<a></li>');
            }
        });
    });
</script>
```
