---
layout: post
title: "[Django-Relationships] 모델관계"
categories: posts
tags: django
---

# [Django-Relationships] 모델관계

## Relationships

+ 관계형 데이터베이스의 강력함은 테이블간의 관계에 있다
+ 장고는 데이터베이스의 관계 유형 중 가장 일반적인 3가지를 제공
	+ `Many-To-One`, `Many-To-Many`, `One-To-One`

### shell 패키지 설치

> shell로 직접 만들어서 db해보기

+ 자동으로 import 해주는 shell 패키지 설치
	+ `pip install django_extensions` 
	+ INSTALLED_APPS 안에 `django_extensions` 넣기
	+ 터미널에서 ./manage.py shell_plus

### on_delete옵션

+ models.CASCADE: 레코드가 삭제되면, 그 레코드를 외래키로 참조하고 있는 모든 레코드들을 함꼐 삭제
+ models.PROTECT: 외래키가 참조하고 있는 레코드를 삭제하지 못하게 만든다. 삭제를 시도하면 `ProtectedError`를 발생시킨다
+ models.SET_NULL: 외래키가 참조하고 있는 레코드가 삭제되면, 외래키 필드의 값이 null값이 된다.
	+ 외래키 필드에 `null=True` 옵션이 있을때만 가능함
+ models.SET_DEFAULT: 외래키가 참조하고 있는 레코드가 삭제되면, 외래키 필드의 값이 기본값을 바뀐다.
	+ `default`옵션이 설정되어 있을 때만 가능함
+ models.SET(): `SET()`함수에 값이나 호출가능한 객체를 전달할 수 있으며, 외래키가 참조하고 있는 레코드가 삭제되면 전달된 값 또는 객체를 호출한 결과로 외래키 필드를 채운다.


### Many-To-One Relationships

+ Many-To-One 관계를 정의하기 위해 `django.db.models.ForeignKey`를 사용
+ 다른 필드 타입과 비슷하게 모델의 클래스속성으로 정의
+ `ForeignKey`는 관계를 정의할 모델 클래스를 인수로 가져야 함

> 예) `Car`모델은 `Manufacturer`를 `ForeignKey`로 갖는다. `Manufacturer`는 여러개의 `Car`를 가질수 있지만 , `Car`는 오직 하나의 `Manufacturer`만을 갖는다.

```
# models.py

from django.db import models


class Manufacturer(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class Car(models.Model):
    manufacturer = models.ForeignKey(
        'Manufacturer',
        on_delete=models.CASCADE,
    )
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name

```

```
# shell로 db할당 해보기

> kia = Manufacturer.objects.create(name='기아')
> hyundia = Manufacturer.objects.create(name='현대')

-> Manufacturer에 기아, 현대를 만들어줌

> Car.objects.create(manufacturer=hyundia, name='아반떼')
> Car.objects.create(manufacturer=hyundia, name='소나타')
> Car.objects.create(manufacturer=hyundia, name='그랜저')

-> Car에 아반떼, 소나타, 그랜저 데이터 만들기, 위 방법으로도 하나씩 데이터를 만들수있지만 리스트컴프리헨션으로 한번에 데이터 만들기를 할수있다 밑에 예시보기

> k3, k5, k7 = [Car.objects.create(manufacturer=kia, name=name) for name in 'k3 k5 k7'.split()]

-> 리스트컴프리헨션으로 리스트객테를 만들고 튜플 언패킹으로 변수에 담아 할당하기
```

### ForeignKey self 

```
# 예) 학생과 강사

class FCUser(models.Model):
    name = models.CharField(max_length=30)
    instructor = models.ForeignKey(
        'self',
        blank=True,
        null=True,
        on_delete=models.SET_NULL,
        # 역참조
        related_name='students',
    )
```

+ related_name='students'
	+ `related_name`을 설정을 안해주면 역참조할때 class명의 소문자(fcuser_set)로 할당

```
# shell로 db에 할당해보기


> FCUser.objects.create(name='박성민')		- pk1
> FCUser.objects.create(name='박성준')		- pk2
> FCUser.objects.create(name='박기환')		- pk3
> FCUser.objects.create(name='이한영')		- pk4
> user4 = FCUser.objects.exclude(name='이한영').update(instructor=FCUser.objects.get(name='이한영'))

-> db 확인해보면 instructor_id에 이한영은 null값을 가지고 박성민(pk1), 박성준(pk2), 박기환(pk3)은 이한영(pk4)를 갖는다

> user4.students.all()

-> 이한영을 참조하고있는 학생들 목록보여줌
박성민, 박성준, 박기환
```



### Many-To-Many Relationships

+ 어느쪽이 우선관계다 테이블 개념에서 존재하지않음
+ 둘의 관계를 중간테이블에서 관리
	+ pizza, topping 테이블이 만들어지면서 중간에 `pizza_topppings` 테이블이 같이 생성됨
+ 의미상에 맞게 필드를 만들어주면 됨
	+ 피자와 토핑을 만든다고 가정한다면 피자위에 토핑이 올라가는게 의미상 맞기떄문에 피자에 필드를 만들어줌


```
# 예) many_to_many 일반예시

from django.db import models

__all__ = (
    'Topping',
    'Pizza',
)


class Topping(models.Model):
    name = models.CharField(max_length=10)

    def __str__(self):
        return self.name


class Pizza(models.Model):
    name = models.CharField(max_length=10)
    toppings = models.ManyToManyField(Topping)

    def __str__(self):
        return self.name
```

```
# shell 

> 치즈피자 = Pizza.objects.create(name='치즈피자')
> 불고기피자 = Pizza.objects.create(name='불고기피자')
> 치즈, 불고기, 피망 = [Topping.objects.create(name=name) for name in '치즈 불고기 피망'.split()]

-> Pizza: 치즈피자, 불고기피자 객체 만들어줌
-> Topping: 치즈, 불고기, 피망 객체 만들어줌

> 치즈피자.toppings.add(치즈)
> 불고기피자.toppings.add(치즈,불고기,피망)

-> 각 피자에 토핑 추가해줌

> 치즈피자.toppings.all()
> 불고기피자.toppings.all()

-> 각 피자에 토핑목록 보기

> 치즈.pizza_set.all()
> 불고기.pizza_set.all()

-> 피자에 토핑 치즈,불고기가 올라간 피자목록보기 (역참조)
```


> 두 모델 사이의 관계와 데이터를 연결할때

+ through : 인수에 중간 모델을 가리키도록 하여 연결 할 수 있다

+ 일반적인 MTM필드가 아니면 add(), create(), set() 직접할당 명령어를 사용할 수 없다


```
예) intermediate.py

from django.db import models

__all__ = (
    'Person',
    'Group',
    'Membership',
)


class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name


class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        # MTM관꼐에 대한 정보를 가질 테이블을 명시적으로 사용
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


### 자기 자신클래스에 관계 가지기

```
# recursive.py
# 어떤 유저로 부터 어떤 유저로 관계가 만들어졌는지
# 양쪽이 동등한관계를 갖음

from django.db import models

__all__ = (
    'FacebookUser',
)


class FacebookUser(models.Model):
    name = models.CharField(max_length=50)
    friends = models.ManyToManyField(
        'self',
    )

    def __str__(self):
        return self.name

```

+ symmetrical=False
	+ 양쪽이 동등하지 않다

```
# recursive_symmetrical_false.py

from django.db import models

__all__ = (
    'InstagramUser',
)


class InstagramUser(models.Model):
    name = models.CharField(max_length=50)
    # 내가 팔로우하는 유저 목록 : following
    # 나를 팔로우하는 유저 목록 : followers
    following = models.ManyToManyField(
        'self',
        symmetrical=False,
        related_name='followers',
    )

    def __str__(self):
        return self.name

```



```
# recursive_symmentrical_false_intermediate.py

from django.db import models
from django.utils import timezone

__all__ = (
    'TwitterUser',
    'Relation',
)


class TwitterUser(models.Model):
    name = models.CharField(max_length=50)
    relation_users = models.ManyToManyField(
        'self',
        through='Relation',
        related_name='+',
        symmetrical=False,
    )

    def __str__(self):
        return self.name

    @property
    def followers(self):
        """
        :return: 나를 follow하는 다른 TwitterUser QuerySet
        """
        return TwitterUser.objects.filter(
            from_user_relations__to_user=self,
            from_user_relations__relation_type='f',
        )

    @property
    def following(self):
        """
        :return: 내가 follow하는 다른 TwiiterUser QuerySet
        """
        return TwitterUser.objects.filter(
            to_user_relations__from_user=self,
            to_user_relations__relation_type='f',
        )

    @property
    def block_list(self):
        """
        :return: 내가 block하는 다른 TwitterUser QuerySet
        """
        return TwitterUser.objects.filter(
            to_user_relations__from_user=self,
            to_user_relations__relation_type='b',
        )

    @property
    def follow(self, user):
        """
        user를 follw하는 Relation을 생성
            1. 이미 존재한다면 만들지 않는다
            2. user가 block_list에 속한다면 만들지 않는다
        :param user: TwitterUser
        :return: Relation instance
        """

        if not self.from_user_relations.filter(to_user=user).exists():
            self.from_user_relations.create(
                to_user=user,
                relation_type='f',
            )

        return self.from_user_relations.get(to_user=user)

    @property
    def block(self, user):
        """
        user를 block하는 Relation을 생성
            1. 이미 존재한다면 만들지 않는다
            2. user가 following에 속한다면 해제시키고 만든다
        :param user: TwitterUser
        :return: Relation instance
        """

        try:
            # Relation이 존재함
            relation = self.from_user_relations.get(to_user=user)
            if relation.relation_type == 'f':
                # 근데 following이라면 block로 바꾸고, 생성일자를 지금 시간으로 변경 후 저장
                relation.relation_type = 'b'
                relation.create_at = timezone.now()
                relation.save()
        except Relation.DoesNotExist:
            # Relation이 없다면 생성 후 생성여부값에 True할당
            relation = self.from_user_relations.create(to_user=user, relation_type='b')

        # Relation인스턴스와 생성여부를 반환
        return relation

    @property
    def follower_relations(self):
        """
        :return: 나를 follow하는 Relation QuerySet
        """
        return self.to_user_relations.filter(relation_type='f')

    @property
    def followee_relations(self):
        """
        :return: 내가 follow하는 Relation QuerySet
        """
        return self.from_user_relations.filter(relation_type='f')


class Relation(models.Model):
    CHOICES_RELATION_TYPE = (
        ('f', 'Follow'),
        ('b', 'Block'),
    )
    from_user = models.ForeignKey(
        'TwitterUser',
        on_delete=models.CASCADE,
        related_name='from_user_relations',
        related_query_name='from_user_relation',
    )
    to_user = models.ForeignKey(
        'TwitterUser',
        on_delete=models.CASCADE,
        related_name='to_user_relations',
        related_query_name='to_user_relation',
    )
    relation_type = models.CharField(choices=CHOICES_RELATION_TYPE, max_length=1)
    create_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        # 두필드에 대해 중복 막음 1개만 맺을수 있도록 설
        unique_together = (
            ('from_user', 'to_user')
        )

    def __str__(self):
        return '{from_user} to {to_user} ({type})'.format(
            from_user=self.from_user.name,
            to_user=self.to_user.name,
            # CHOICE가 정의되어있는 CharField
            # get_FOO_display() <- FOO:Field
            type=self.get_relation_type_display(),
        )

```


### Ont-To-One

한 테이블의 하나의 레코드가 다른 테이블의 단 하나의 레코드만을 참조할 때, 이 두 모델간의 관계를 일대일 관계 라고 한다. 일대일 관계는 어떤 테이블을 구조적으로 확장시킬 때 가장 유용하게 쓰인다.
다음의 예를 보자.

`Django` 에서 모델에 일대일 관계 필드를 추가하려면 `OneToOneField` 를 사용한다.
`OneToOneField` 는 `ForignKey` 필드에 `unique=True` 옵션을 준 것과 동일하게 동작한다. 즉, 외래키 필드의 값은 반드시 고유한 값이어야 한다.
재귀적 관계와 아직 정의되지 않은 관계 또한 `ForignKey` 필드와 동일한 방식으로 설정한다.

+ 일대일 관계는 다른 모델을 확장하여 새로운 모델을 만드는 경우 유용하게 사용

+ hasattr(p2, 'restaurant')
	+ 속성이 존재하는지 확인 할수있다
	
```
# one-to-one

from django.db import models


class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

    def __str__(self):
        return f'{self.name} the place'


class Restaurant(models.Model):
    place = models.OneToOneField(
        Place,
        on_delete=models.CASCADE,
        primary_key=True,
    )
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    def __str__(self):
        return f'{self.place.name} the restaurant'


class Waiter(models.Model):
    restaurant = models.ForeignKey(
        Restaurant,
        on_delete=models.CASCADE,

    )
    name = models.CharField(max_length=50)

    def __str__(self):
        return '{name} the waiter at {restaurant}'.format(
            name=self.name,
            restaurant=self.restaurant,
        )
```

### Abstract base classes

+ 추상 기본 클래스는 몇 가지 공통된 정보를 여러 다른 모델에 넣으려 할때 유용
+ 기본 클래스를 작성하고 `Meta` class에 `abstract = True`를 넣는다
+ 이 모델은 데이터베이스 테이블을 만드는데 사용되지 않음
+ 다른 모델의 기본 클래스로 사용될 때 해당 필드는 자식 클래스의 필드에 추가
+ 자식의 이름과 같은 이름(상속받은 클래스의 이름과 같은 이름의 필드)을 가진 추상 기본 클래스의 필드를 갖는 것은 오류이며, `Django`는 이에 대해 오류를 발생
+ 인스턴스화하거나 저장할수 없음

```
# part 1
# 1. AbstractBaseClasses
#     자식 테이블만 존재
# 2. Multi table inheritance
#     보무, 자식 테이블이 모두 존재
# 3. Proxy model
#     부모 테이블만 존재

from django.db import models


class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True


class Student(CommonInfo):
    home_group = models.CharField(max_length=5)

```

### 상속

```
part 2
# 예제 (abstract_base_classes/ models.py)

from django.db import models

__all__ = (
    'RelatedUser',
    'PhotoPost',
    'TextPost',
    'PostBase',
)


class RelatedUser(models.Model):
    name = models.CharField(max_length=50)

    def __str__(self):
        return self.name


class PostBase(models.Model):
    author = models.ForeignKey(
        RelatedUser,
        on_delete=models.CASCADE,
        # 유저(Person) 입장에서
        # 자신이 특정 Post의 author인 경우에 해당하는 모든 PostBase 객체를 참조
        # %(class)s : 상속받은 클래스명의 소문자화
        # %(app_label)s : 상속받은 클래스가 속한 애플리케이션명의 소문자화
        related_name='%(app_label)s_%(class)s_set',
        related_query_name='%(app_label)s_%(class)s'

    )
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        abstract = True


class PhotoPost(PostBase):
    # author의 related_name
    # photopost_set
    photo_url = models.CharField(max_length=100, blank=True)

    def __str__(self):
        return f'Post (author: {self.author.name})'


class TextPost(PostBase):
    # author의 related_name
    # textpost_set
    text = models.TextField(blank=True)

    def __str__(self):
        return f'Post (author: {self.author.name})'

```

```
part 2
# abc_other/ models.py

from django.db import models
from abstract_base_classes.models import PostBase


class PhotoPost(PostBase):
    pass

```

+ %(class)s : 상속받은 클래스명의 소문자화
+ %(app_label)s : 상속받은 클래스가 속한 애플리케이션명의 소문자화
+ 다른 애플리케이션 모델 클래스에서 상속받을때 겹치지 않기위해 사용

> `related_name='%(app_label)s_%(class)s_set'`
> `related_query_name='%(app_label)s_%(class)s'` 


### Multi_Table

+ 간단하게 상속받는 테이블사용시 사용
+ 상속받는 테이블이 많아질수록 느림
+ 웬만하면 사용안함

```
예) multi_table/models.py

from django.db import models


class Place1(models.Model):
    # 암시적으로 생성되는 PK Field
    # id = models.AutoField(Primary_key=True)
    # -> 임의의 필드에 primary_key=True 옵션을 주면 id 필드가 생성되지 않음
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80, blank=True)

    def __str__(self):
        return f'{self.name} [Place]'

    class Meta:
        db_table = 'Inheritance_MultiTable_Place'


def get_removed_place():
    # 밑에 내용줄여서 쓰면 return Place1.objects.get_or_create(name='철거됨')[0]
    try:
        place = Place1.objects.get(name='철거됨')
    except Place1.DoesNotExist:
        place = Place1.objects.create(name='철거됨')
    return place


class Restaurant1(Place1):
    # MultiTable inheritance구현 시 암시적으로 생성되는 OTO Field
    # <부모클래스의 소문자화>_ptr = models.OneToOneField(<부모클래스>)
    # PLace1_ptr = models.OneToOneField(Place1, primary_key=True)
    #   -> 임의의 필드에 parent_link=True 옵션을 주면 <부모클래스의 소문자화>_ptr 필드가 생성되지 않음
    place_ptr = models.OneToOneField(
        Place1,
        on_delete=models.CASCADE,
        parent_link=True,
        primary_key=True,
    )

    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    # OTO으로 구현할 내용 아님
    old_place = models.ForeignKey(
        Place1, verbose_name='이전에 가게가 있던 장소',
        # 만약에 이전에 가게가 있던 건물이 없어질 경우
        # (해당 장소 또는 건물이 없어졌음) 이라는 정보를 담자
        # -> 위와 같은 정보를 담고 있는 Place1객체 (DB row)가 필요함
        on_delete=models.SET(get_removed_place),
        # Place목록 중에서, 자신이 old_place인 경우에 해당하는 Restaurant목록
        # -> 즉, 자신(장소)에 있었던 Restaurant목록 -> (그 장소에) 예전에 있던 식당 목록
        related_name='old_restaurants',
        related_query_name='old_restaurant',
        blank=True,
        null=True
    )

    def __str__(self):
        return f'{self.name} [Restaurant]'

    class Meta:
        db_table = 'Inheritance_MultiTable_Restaurant'
```

+ `Place1_ptr, old_place`의 `query_name`이 출동하여 `related_query_name` 설정
	+ `Place1_ptr(OTO)`, `old_place(MTOM)`
+ parent_link=True
	+ multi_table 상속을 받았을때 부모와의 연결을 정의하는 필드를 나타내기위해 사용
	+ OTO에서만 쓰임


### Proxy(대리) models

+ 부모테이블만 생성 자식테이블은 생성안됨.
+ 기본 관리자를 변경하거나 새 메서드를 추가하는 경우 해당
+ 원본 모델에 대한 proxy를 만듬
	+ 프록시 모델의 인스턴스를 생성, 삭제 및 업데이트 할수 있음
	+ 원본(비 프록시) 모델을 사용하는 것처럼 모든 데이터가 저장됨.
+ 차이점
	+ 원본을 변경하지 않고 프록시의 기본 모델 순서(ordering)또는 기본 관리자(default manager)와 같은 것을 변경할 수 있다는 것
+ 프록시 모델은 일반 모델처럼 선언됨
+ Django에게 메타 클래스의 proxy속성을 True로 설정하여 Django에게 해당 모델이 프록시 모델임을 알려야함


```
# 예

from django.db import models


class User1Manager(models.Manager):
    def nomal_users(self):
        # super().get_queryset()
        # 상위 클래스에서 정의한 '기본적으로' 돌려줄 QuerySet
        return super().get_queryset().filter(is_admin=False)
        # return User1.objects.filter(is_admin=False)

    def admin_users(self):
        return super().get_queryset().filter(is_admin=True)


class User1(models.Model):
    name = models.CharField('이름', max_length=40)
    is_admin = models.BooleanField('관리자', default=False)

    # 이 클래스에 커스텀 매니저를 적용
    objects = User1Manager()

    def __str__(self):
        return self.name

    class Meta:
        db_table = 'Inheritance_Proxy_User1'

    def find_user(self, name):
        return User1.objects.filter(name__contains=name)


class NormalUserManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_admin=False)


class AdminUserManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_admin=True)


class NomalUser(User1):
    objects = NormalUserManager()

    class Meta:
        proxy = True


class Admin(User1):
    objects = AdminUserManager()

    class Meta:
        proxy = True

    def delete_user(self, user):
        user.delete()
```

+ 의미에 따라 클래스를 나눠쓰는데 테이블은 하나인데 두종류의 클래스를 쓰는것처럼 다룰수 있음
+ 개발자가 실수하는것을 막을수있음
+ 특정모델클래스에서 필드하나로 클래스 인스턴스의 타입이 결정된다면 일일이 필터를 가져오는게아니라 필터를 가지고 구분할 객체들을 각각 proxy모델로 만들어주는것이 사용성이 좋음



참조 - 역참조 요약  |OneToOneField|ManyToOneField|ManyToManyField
---|---|---|---|---|
참조	|	source.attname|source.attname|source.attname
(소스 > 타켓)|단일 객체 리턴| 단일 객체 리턴 | 다수의 객체 리턴
역참조| target.lowersource| target.lowersource_set| target.lowersource_set
(타겟 >. 소스)| 단일 객체 리턴 | 다수의 객체 리턴 | 다수의 객체 리턴
 
 
