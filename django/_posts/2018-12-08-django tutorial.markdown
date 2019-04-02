---
layout: post
title: "[Django-tutorial] 장고튜토리얼 사용해보기"
categories: posts
tags: django
---

# [Django-tutorial] 장고 튜토리얼 사용해보기



## 초기설정

```
django_tutorial폴더 생성 -> git,gitignore설정 
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


### app urls 사용하기

+ app 별로 urls 나누기
+ config/ urls.py에 include하기
	+ `예) path('polls/', include('polls.urls')),`

```
# polls/ urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),

]
```

+ path('')로 접근시 곧바로 'index' URL(polls.views.index에 해당)로 이동하도록 config.views.index 구현

```
# config/ urls.py

from . import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
    path('', views.index),   <- 추가
]


# config/ views.py

from django.shortcuts import redirect


def index(request):
    # 곧바로 /polls/로 redirect되도록 설정
    return redirect('index')

```

+ appname (namespace) 사용
	+ 여러 프로젝트할시 name이 겹칠수도 있어 사용
	
```
# polls/urls.py 

from django.urls import path
from . import views


app_name = 'polls'.   <--- 추가

urlpatterns = [
    path('', views.index, name='index'),
    # r'^(?P<question_id>\d+)/$'
    path('<int:question_id>/', views.detail, name="detail"),
    path('<int:question_id>/results/', views.results, name="results"),
    path('<int:question_id>/vote/', views.vote, name="vote")
]

```




### INSTALLED_APPS에 app추가, 모델생성

```
# settings.py

INSTALLED_APPS = [
    'polls.apps.PollsConfig',    <- 추가
]

# 모델 생성(polls/models.py)

from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('게시일자')
	
	def __str__(self):
    	return self.question_text

	def was_published_recently(self):
        """
        최근에 게시되었는지 여부를 리턴
        게시 한지 1일 이내인지 여부를 리턴

        return 자신의 게시일자 >= 지금 - 1일
            이 식이 참이다
                -> 24시간이 지나지 않았다. 게시 일자가
                -> 게시한 지 1일 미만인 상태이다.
            거짓이다
                -> 게시한 지 1일이 지났다.
        :return:
        """
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    question = models.ForeignKey(
        Question,
        on_delete=models.CASCADE,
    )
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

	
    def __str__(self):
        return self.choice_text

```


### View 사용하기

+ 파일 패키지화 만들기(views.py 마우스 우클릭 -> Refactor -> Convert to Python Package

```
# polls/views.py

def index(request):
    # Question 클래스에 대한 QuerySet 을 가져옴
    # 게시일자 속성에 대한 내림차순 순서로 최대 5개까지
    latest_question_list = Question.objects.order_by('-pub_date')[:5]

    context = {
        'latest_question_list': latest_question_list,
    }

    return render(request, 'polls/index.html', context)


def detail(request, question_id):
    # try:
    #     question = Question.objects.get(pk=question_id)
    # except Question.DoesNotExist:
    #     raise Http404('Question does not exist')
    question = get_object_or_404(Question, pk=question_id)
    context = {
        'question': question,
    }
    return render(request, 'polls/detail.html', context)


def results(request, question_id):
    context = {
        'question': get_object_or_404(Question, pk=question_id)
    }
    return render(request, 'polls/result.html', context)


def vote(request, question_id):
    # question_id가 pk인 Question객체를 DB로부터 가져온 데이터로 생성
    # 만약 해당하는 Question이 없다면 Http404예외가 발생함
    question = get_object_or_404(Question, pk=question_id)
    try:
        # 현재 투표중인 Question에 속한 Choice목록에서
        # pk가 POST요청에 전달된 'choice'값에 해당하는 Choice객체를 selected_choice변수에 할당
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # 위의 try문에서 발생할 수 있는 예외는 2가지
        # 1. KeyError: request.POST에 'choice'키가 없을 때 발생
        # 2. Choice.DoesNotExist: question.choice_set.get(pk=무언가)에서 발생
        # (pk에 해당하는 객체가 DB에 없을 경우)
        context = {
            'question': question,
            'error_message': "You didn't select a choice.",
        }
        return render(request, 'polls/detail.html', context)
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return redirect('polls:results', question_id)



```

### templates 사용

```
# polls/templates/polls/index.html

{% raw %}{% if latest_question_list %}
    <ul>
        {% for question in latest_question_list %}
        <li>
            <a href="{% url 'polls:detail' question.id %}">
                {{ question.question_text }}
            </a>
        </li>
        {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
    {% endif %}{% endraw %}
```

```
# polls/templates/polls/detail.html

<h1>{{ question.question_text }}</h1>
{% raw %}
{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="POST">
    {% csrf_token %}
    {% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">
        {{ choice.choice_text }}
    </label><br>
    {% endfor %}{% endraw %}
    <input type="submit" value="Vote">
</form>


```

```
# polls/templates/polls/result.html

<h1>{{ question.question_text }} Results</h1>
<ul>
{% raw %}
    {% for choice in question.choice_set.all %}
        <li>
            {{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize}}
        </li>
    {% endfor %}
</ul> 

<a href="{% url 'polls:detail' question.pk %}">Vote again</a>
<a href="{% url 'polls:index' %}">Return to polls index</a>
{% endraw %}
```


### clsss generic View사용하기

```
# polls/urls.py

from django.urls import path, include

from . import views
from .views import cbv,fbv as views


app_name = 'polls'

urlpatterns_cbv = [
    path('', cbv.IndexView.as_view(), name='index'),
    path('<int:question_id>/', cbv.DetailView.as_view(), name='detail'),
    path('<int:question_id>/vote/', cbv.VoteView.as_view(), name='vote'),
]

urlpatterns = [
    path('', views.index, name='index'),
    # r'^(?P<question_id>\d+)/$'
    path('<int:question_id>/', views.detail, name="detail"),
    path('<int:question_id>/results/', views.results, name="results"),
    path('<int:question_id>/vote/', views.vote, name="vote"),

    # /polls/cbv/로 시작하는 URL요청은
    # 위의 urlpatterns_cbv리스트 내의 내용에서 처리
    path('cbv/', include(urlpatterns_cbv)),
]

```

```
# polls/views.py

# polls.views.cbv(class-based view)
from django.shortcuts import get_object_or_404, render, redirect
from django.views import generic
from django.views.generic.base import View

from ..models import Question, Choice


class IndexView(generic.ListView):
    model = Question
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'
    ordering = ('-pub_date',)

    def get_queryset(self):
        return super().get_queryset()[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'
    pk_url_kwarg = 'question_id'


class VoteView(View):
    def post(self, request, question_id):
        # question_id가 pk인 Question객체를 DB로부터 가져온 데이터로 생성
        # 만약 해당하는 Question이 없다면 Http404예외가 발생함
        question = get_object_or_404(Question, pk=question_id)
        try:
            # 현재 투표중인 Question에 속한 Choice목록에서
            # pk가 POST요청에 전달된 'choice'값에 해당하는 Choice객체를 selected_choice변수에 할당
            selected_choice = question.choice_set.get(pk=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            # 위의 try문에서 발생할 수 있는 예외는 2가지
            # 1. KeyError: request.POST에 'choice'키가 없을 때 발생
            # 2. Choice.DoesNotExist: question.choice_set.get(pk=무언가)에서 발생
            # (pk에 해당하는 객체가 DB에 없을 경우)
            context = {
                'question': question,
                'error_message': "You didn't select a choice.",
            }
            return render(request, 'polls/detail.html', context)
        else:
            selected_choice.votes += 1
            selected_choice.save()
            return redirect('polls:results', question_id)

```


### Test 코드짜기

```
# polls/test.py

import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        # 미래의 datetime객체를 time변수에 해당
        time = timezone.now() + datetime.timedelta(days=30)
        # 새 Question인스턴스를 생성, pub_date값에 미래시간을 주어줌
        future_question = Question(pub_date=time)
        # 주어진 2개의 객체가 같아야 할것 (is)으로 기대함
        # 같지 않으면 실패
        self.assertIs(
            future_question.was_published_recently(),
            False,
        )


```
