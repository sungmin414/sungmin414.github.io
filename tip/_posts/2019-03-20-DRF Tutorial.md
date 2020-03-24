---
layout: post
title: "[DRF] Tutorial"
categories: tip
tags: tip
---

# DRF Tutorial

## Postman

+ [Postman사이트](https://www.getpostman.com/)
+ 개발한 API 테스트하고 결과를 공유하여 API 개발의 생산성을 높여주는 플랫폼이다

```
# GET요청 할 주소를 치면 내용을 볼수있지만 반복적인 작업을할때 설정해주기

Collections(이름정해주고 create)에 폴더 추가하기 -> 주소입력하고 save 누르기

-> Request name정해주고 사용할 Collection 폴더 선택 후 save

```


## REST API 사용법

+ [REST API 제대로 알고 사용하기!! (한번 읽어보면 좋음)](https://meetup.toast.com/posts/92)

## DRF Tutorial

### Serialization

+ 웹 API만들어 보기
+ REST 프레임 워크 다양한 기능 해보기
+ [DRF3 한글문서](http://raccoonyy.github.io/drf3-tutorial-1.html)
+ [django-rest-framework공식문서](https://www.django-rest-framework.org/)



```
> settings 순서

pipenv --python 3.6.8 -> pipenv shell (해당 가상환경적용) 

-> pipenv install django djangorestframework pygments

-> django-admin startproject config(mv config app)

-> interpreter설정(~/.local/share/virtualenvs/<가상환경명>)

-> ./manage.py startapp snippets

```

> 개발환경에서 사용할 패키지설치는 --dev를 마지막에 붙인다. 

> 예)`pipenv install <패키지명> --dev`

+ `INSTALLED_APPS`에 `rest_framework` 추가해주기
 
### DRF에 필요한 패키지 설치

+ `ipython(개발환경에만 설치 --dev)`, `django_extensions(INSTALLED_APPS에 추가해주기)`

### Snippets app

+ Serializer사용해보기
+ 웹 API를 만들려면 우선 Snippet 클래스의 인스턴스를 json 같은 형태로 직렬화(serializing)하거나 반직렬화(deserializing)할 수 있어야 한다
+ Django REST 프레임워크에서는 Django 폼과 비슷한 방식으로 시리얼라이저를 작성

```
> snippets/models.py

from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)


```

```
> snippets/serializers.py

from rest_framework import serializers
from .models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    pk = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        검증한 데이터로 새 'Snippet' 인스턴스를 생성하여 리턴
        :param validated_data:
        :return:
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        검증한 데이터로 기존 'Snippet' 인스턴스를 업데이트한 후 리턴
        :param instance:
        :param validated_data:
        :return:
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.save()
        return instance
```

+ Snippet클래스의 인스턴스를 만들어보고, 직렬화해보기

```
> ./manage.py shell_plus

# snippet 인스턴스 만들기

from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

s1 = Snippet(code='foo = "bar"\n')
s1.save()

s2 = Snippet(code='print "hello, world"\n')
s2.save()


# 인스턴스 직렬화하기(python 데이터 타입으로 변환)

serializer = SnippetSerializer(s1)
serializer.data


# 직렬화 과정 마무리 하려면 데이터를 json으로 변환하기

content = JSONRenderer().render(serializer.data)
content


# 반직렬화도 비슷함, 파이썬 데이터 타입을 파싱하기

import io

stream = BytesIO(content)
data = JSONParser().parse(stream)


# 데이터 인스턴스화하기

serializer = SnippetSerializer(data=data)
serializer.is_vlid()	유효성 검사
serializer.validated_data
serializer.save()


# 모델의 인스턴스 말고도 쿼리셋도 직렬화 할수 있음(many=True 추가)
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data

```

+ 객체를 json화 시켜서 텍스트파일에 저장하기
	+ 객체의 속성들만 딕셔너리로 꺼내와서 json형식으로 저장
	
```
# 두가지 방법
> ./manage.py shell_plus

from jdnaog.forms.models import model_to_dict

# 1
f = open('s1.txt', 'wt')
json.dump(model_to_dict(s1), f)
f.close()

# 2 
with open('s1.txt', 'wt') as f:
	json.dump(model_to_dict(s1), f)
```




## ModelSerializer, Postman

+ ModelSerializer
	+ ModelSerializer는 필드를 자동인식함
	+ create()메서드와, update()메서드가 이미 구현되어있음

```
from rest_framework import serializers
from .models import Snippet


class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = (
            'pk',
            'title',
            'code',
            'linenos',
            'language',
            'style',
        )

```

+ 파이썬 기본자료형 바꾸기

```
# 객체를 불러왔을때 확인해보기 (TEST)

from snippets.serializers import SnippetSerializer

s1 = Snippet.objects.first()
serializer = SnippetSerializer(s1)
serializer.data


# json 형식으로 데이터바꾸기

import json

json.dump(serializer.data, open('s1.txt', 'wt'))

# 문자열 데이터를 읽고 json형식으로 파싱하기
# json문자열을 파이썬 객체로 바꿈
# 역직렬화하면 pk는 빠짐

data = json.load(open('s1.txt', 'rt'))

serializer = SnippetSerializer(data=data)
serializer.is_valid()
serializer.data
serializer.save()

```

+ 시리얼라이저를 사용하는 Django view
	+ 인증되지 않은 사용자도 뷰에 POST할수 있도록 `csrf_exempt` 데코레이터 사용

```
> snippets/views/django.fbv.py

from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.parsers import JSONParser

from .serializers import SnippetSerializer
from .models import Snippet

# CSRF검증에서 제외되는 view
@csrf_exempt
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        # Snippet QuerySet을 생성자로 사용한 SnippetSerializer인스턴스
        serializer = SnippetSerializer(snippets, many=True)
        # JSON형식의 문자열을 HttpResponse로 돌려줌 (content_type에 'application/json'명시됨)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        # request를 분석해서 전달받은 JSON형식 문자열을 파이썬 데이터형으로 파싱
        data = JSONParser().parse(request)
        # data인수를 채우면서 Serializer인스턴스 생성 (역직렬화 과정)
        serializer = SnippetSerializer(data=data)
        # Serializer의 validation
        if serializer.is_valid():
            # valid한 경우, Serializer의 save()메서드로 새 Snippet인스턴스 생성
            serializer.save()
            # 생성 후 serializer.data로 직렬화한 데이터를 JSON형식으로 리턴 201(Created)상태코드 전달
            return JsonResponse(serializer.data, status=201)
        # invlid한 경우, error목목을 JSON형식으로 리턴하며 400(bad Request)상태코드 전달
        return JsonResponse(serializer.errors, status=400)
        
@csrf_exempt
def snippet_detail(request, pk):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        # Serializer인스턴스 생성에 모델 클래스 인스턴스와 data를 함께 사용 
        # ( 전달받은 데이터로 인스턴스의 내용을 update예정 )
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == "PATCH":
        data = JSONParser().parse(request)
        # Partial=True
        # Serializer의 required=True 필드의 값이 주어지지 않아도 valid 하다고 판단
        serializer = SnippetSerializer(snippet, data=data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == "DELETE":
        snippet.delete()
        return HttpResponse(status=204)

```

+ api view데코레이터 사용

```
> views/drf_fbv.py

from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response

from ..models import Snippet
from ..serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    if request.method == 'GET':
        snippet = Snippet.objects.all()
        serializer = SnippetSerializer(snippet, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PATCH', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method in ('PATCH', 'PUT'):
        serializer = SnippetSerializer(snippet, data=request.data, partial=(request.method == 'PATCH'))
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

### 브라우저로 xml, yaml방식으로 보기
+ 패키지 설치
	+ `pipenv install djangorestframework-yaml`
	+ `pipenv install djangorestframework-xml`


```
# settings.py안에 추가

# DRF(render, parser, )
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
        'rest_framework_yaml.parsers.YAMLParser',
        'rest_framework_xml.parsers.XMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
        'rest_framework_yaml.renderers.YAMLRenderer',
        'rest_framework_xml.renderers.XMLRenderer',
    ),

}
```
```
> views.py 안에 매개변수로 format=None 넣어줌
```

```
# snippets/urls/drf_fbv.py안에 추가

from rest_framework.urlpatterns import format_suffix_patterns

urlpatterns = format_suffix_patterns(urlpatterns)

```
+ 위 내용을 적용한후 `localhost:8000/drf-fbv/snippets`로 들어가면 브라우저 방식으로 내용을 볼수있음



## 클래스 기반 뷰


```
> urls/drf_cbv.py

from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns

from ..views import drf_cbv as views


urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

```
> views/drf_cbv.py

from django.http import Http404
from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView

from ..serializers import SnippetSerializer
from ..models import Snippet


class SnippetList(APIView):
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class SnippetDetail(APIView):
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None, **kwargs):
        partial = kwargs.pop('partial', False)

        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data, partial=partial)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def patch(self, request, pk, format=None):
        # snippet = self.get_object(pk)
        # serializer = SnippetSerializer(snippet, data=request.data, partial=True)
        # if serializer.is_valid():
        #     serializer.save()
        #     return Response(serializer.data)
        # return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        return self.put(request, pk, format, partial=True)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```



## 믹스인(Mixins)

+ 클래스 기반 뷰를 사용하여 얻는 가장 큰 이점은 기능들을 손쉽게 조합할 수 있다.


```
> urls/mixins.py

from django.urls import path
from ..views import drf_mixin as views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]

```

```
> views/drf_mixin.py

from rest_framework import mixins, generics
from ..models import Snippet
from ..serializers import SnippetSerializer


class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)


class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)

```

## 제네릭 클래스 기반 뷰

+ 믹스인 클래스를 사용하여 뷰의 코드를 많이 줄였지만 더 줄일 수 있음
+ REST 프레임 워크에서 믹스인과 연결된 제네릭 뷰를 제공

```
> urls/drf_generic_cbv.py

from django.urls import path
from ..views import drf_generic_cbv as views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]
```

```
> views/drf_generic_cbv.py

from rest_framework import generics
from ..models import Snippet
from ..serializers import SnippetSerializer


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

```

