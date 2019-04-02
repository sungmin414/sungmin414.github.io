---
layout: post
title: "[Django-Relationships] 모델관계"
categories: posts
tags: django
---

# [Django-Relationships] 모델관계

+ 관계형 데이터베이스 
	+ many-to-one
	+ many-to-many
	+ one-to-one


### on_delete옵션

+ models.CASCADE: 레코드가 삭제되면, 그 레코드를 외래키로 참조하고 있는 모든 레코드들을 함꼐 삭제
+ models.PROTECT: 외래키가 참조하고 있는 레코드를 삭제하지 못하게 만든다. 삭제를 시도하면 `ProtectedError`를 발생시킨다
+ models.SET_NULL: 외래키가 참조하고 있는 레코드가 삭제되면, 외래키 필드의 값이 null값이 된다.
	+ 외래키 필드에 `null=True` 옵션이 있을때만 가능함
+ models.SET_DEFAULT: 외래키가 참조하고 있는 레코드가 삭제되면, 외래키 필드의 값이 기본값을 바뀐다.
	+ `default`옵션이 설정되어 있을 때만 가능함
+ models.SET(): `SET()`함수에 값이나 호출가능한 객체를 전달할 수 있으며, 외래키가 참조하고 있는 레코드가 삭제되면 전달된 값 또는 객체를 호출한 결과로 외래키 필드를 채운다.


### Many-to-one

+ Many-to-one 관계를 정의하기위해, `django.db.models.ForeignKey`를 사용
+ 다른 필드 타입과 비슷하게 모델의 클래스속성으로 정의
+ `ForeignKey`는 관계를 정의할 모델 클래스를 인수로 가져야함

예를 들어 `Car`모델은 `Manufacturer`를 `ForeignKey`로 갖는다. `Manufacturer`는 여러개의 `Car`를 가질수 있지만 `Car`는 오직 하나의 `Manufacturer`만을 갖는다.

```
# 예)

from django.db import models


class Manufacturer(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name

```


### Many-To-Many

`many-to-many`관계를 정의하기 위해서는, `ManyToManyField`를 사용합니다.
`ManyToManyField`는 관계를 정의할 모델 클래스를 인수로 가져야 합니다.

예를 들어, `Pizza`모델은 여러 개의 `Topping`객체를 가질 수 있습니다. `Topping`은 여러개의 `Pizza`에 올라갈 수 있으며, `Pizza`역시 여러개의 `Topping`을 가질 수 있습니다. 이것은 다음과 같이 나타낼 수 있습니다.

```
# 예)

from django.db import models

__all__ = (
    'Topping',
    'Pizza',
)


class Topping(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class Pizza(models.Model):
    name = models.CharField(max_length=50)
    toppings = models.ManyToManyField(Topping)

    def __str__(self):
        return self.name

```
+ 어떤 모델이 `ManyToManyField`를 갖는지는 중요하지 않지만, 오직 관계되는 둘 중 하나의 모델에만 존재해야합니다.
+ 일반적으로, `ManyToManyField`인스턴스는 form에서 수정할 객체에 가까워야 합니다. 위의 예제에서는 `toppings`가 `Pizza`에 있는 것이 `Topping`이 `pizzas`를 갖는 것보다는 자연스럽기 때문에, `Pizzaform`에서 사용자는 `toppings`를 선택할 가능성이 높습니다.



### Extra fields on many-to-many

+ 두 모델 사이의 관계와 데이터를 연결해야 할수도 있음
+ 예를 들어, 음악가가 속한 음악 그룹을 트래킹(추적)하는 경우를 고려해봅니다. 사람과 그룹으로 멤버를 이루는 관계에서, `ManyToManyField`로 이 관계를 나타내고자 합니다. 하지만, 어떤 사람이 그룹으로 가입할 때 가입하는 날짜와 같은 세부사항들이 추가로 존재합니다.

+ 이런 경우, 장고에서는 `many-to-many`관계를 관리하는 데 사용되는 모델을 지정할 수 있습니다. 그리고 중간 모델에 추가 필드를 넣을 수 있습니다. `ManyToManyField`의 `through` 인수에 중간 모델을 가리키도록 하여 연결할 수 있습니다. 음악가 예제에서는, 다음과 같이 나타냅니다.

```
# 예)

from django.db import models


class Person(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class Group(models.Model):
    name = models.CharField(max_length=50)
    members = models.ManyToManyField(
        Person,
        
        # MTM관계에 대한 정보를 가질 테이블을 명시적으로 사용
        through='Membership',
        related_name='group_set',
        related_query_name='group',
    )

    def __str__(self):
        return self.name


class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)

    def __str__(self):
        return '{} (Group: {})'.format(
            self.person.name,
            self.group.name,
        )
```


### Ont-To-One

한 테이블의 하나의 레코드가 다른 테이블의 단 하나의 레코드만을 참조할 때, 이 두 모델간의 관계를 일대일 관계 라고 한다. 일대일 관계는 어떤 테이블을 구조적으로 확장시킬 때 가장 유용하게 쓰인다.
다음의 예를 보자.

`Django` 에서 모델에 일대일 관계 필드를 추가하려면 `OneToOneField` 를 사용한다.
`OneToOneField` 는 `ForignKey` 필드에 `unique=True` 옵션을 준 것과 동일하게 동작한다. 즉, 외래키 필드의 값은 반드시 고유한 값이어야 한다.
재귀적 관계와 아직 정의되지 않은 관계 또한 `ForignKey` 필드와 동일한 방식으로 설정한다.

```
# 예)

class Place(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=100)


class Restaurant(models.Model):
    place = models.OneToOneField(Place)
    menu = models.CharField(max_length=50, blank=True, null=True)
    rating = models.FloatField(default=0, blank=True, null=True)
```



참조 - 역참조 요약  |OneToOneField|ManyToOneField|ManyToManyField
---|---|---|---|---|
참조	|	source.attname|source.attname|source.attname
(소스 > 타켓)|단일 객체 리턴| 단일 객체 리턴 | 다수의 객체 리턴
역참조| target.lowersource| target.lowersource_set| target.lowersource_set
(타겟 >. 소스)| 단일 객체 리턴 | 다수의 객체 리턴 | 다수의 객체 리턴
 
 
