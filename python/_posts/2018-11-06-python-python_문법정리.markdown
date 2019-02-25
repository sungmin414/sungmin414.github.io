---
layout: post
title: "[python] python-문법정리"
categories: posts
tags: python
---

# [python] 문법정리


파이썬은 모든것 (정수,문자열,함수)이 객체(object)로 이루어져 있다.
객체는 데이터의 형태를 결정해주는 타입으로, 파이썬에서는 객체의 타입을 바꿀 수없다.

+ 변수는 100이란 데이터를 직접 가지는 것이 아니며, 100이라는 정수형 객체가 있고 이 객체를 참조만 하는것


## 숫자

연산자|설명|예|결과
---|---|---|---|
\+	| 더하기		| 32 + 7	| 39 
\-	| 빼기		| 82 - 2	| 80
\*	| 곱하기		| 3 * 7	| 21
/	| 나누기		| 7 / 2	| 3.5
//	| 정수나누기	| 7 // 2	| 3
%	| 나머지		| 7 % 3	| 1
**	| 지수		| 2**10	| 1024



## 진수(base)

기본적으로 숫자형 데이터는 10진수로 간주되지만, 파이썬에서는 2진수, 8진수, 16진수를 표현할 수 있다.

- 2진수(binary): **0b**또는 **0B**로 시작
- 8진수(octal): **0o**또는 **0O**로 시작
- 16진수(hex): **0x**또는 **0X**로 시작


## 형변환(정수, 부동소수)

내장함수 int, float 사용


## 문자열


문자열 사용시('',"", '''''')

+ 문자열 형변환(str)

### 이스케이프 문자

특수문자, 또는 특별한 역할을 하는 의미를 나타내는 문자를 뜻한다.  

이스케이프 문자|설명
---|---
\a	| 비프음 발생
\t	| 탭(tab)
\n	| 줄바꿈
\\\	| \\(역슬래시) 입력
\\'	| 작은따옴표(') 입력
\\"	| 큰따옴표(") 입력


### 인덱스 연산

문자열에서 문자를 추출하기 위해 대괄호와 오프셋을 지정할 수 있다.
가장 왼쪽은 0이며, 가장 오른쪽은 -1로 시작한다.


### 슬라이스 연산

[start:end:step] 형식을사용

- `[:]`
	- 처음부터 끝까지
- `[start:]`
	- start오프셋부터 마지막까지
- `[:end]`
	- 처음부터 end오프셋까지
- `[start:end]`
	- start오프셋부터 end오프셋까지
- `[start:end:step]`
	- start오프셋부터 end오프셋까지, step만큼씩 뛰어넘은 부분


### 길이
내장함수 len을 사용


#### 문자열 나누기(split)

문자열의 내장함수 **split**을 이용

**split**함수에 인자로 주어진 **구분자**를 기준으로 하나의 문자열을 리스트 형태로 반환해준다.

```python
>>> girlsday = "민아,유라,소진,혜리"
>>> girlsday.split(',')
['민아', '유라', '소진', '혜리']
```


**split**함수에 인자를 주지 않을 경우, 공백문자를 구분자로 사용한다.


### 문자열 결합(join)

**split** 함수와 반대의 역할을 한다.  
문자열 리스트를 하나의 문자열로 결합해주며, 각 문자열을 결합해줄 구분자 문자열을 지정한 후 **join**함수를 사용해준다.

```python
>>> girlsday_list = girlsday.split(',')
>>> girlsday_str = ', '.join(girlsday_list)
>>> print(girlsday_str)
민아, 유라, 소진, 혜리
```

### 대소문자 다루기

capitalize() : 맨앞 대문자

title() : 각단어의 첫글자 대문자

upper() : 문자열 모두 대문자

lower() : 문자열 모두 소문자

swapcase() : 대문자는 소문자로, 소문자는 대문자로

### 공식문서

[Text Sequence - String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods)



### 문자열 포맷

#### 옛 스타일 (%)

```
string % data
```

변환타입|설명
---|---
%s	|	문자열
%d	|	10진수
%x	|	16진수
%o	|	8진수
%f	|	10진 부동소수점수
%e	|	지수로 나타낸 부동소수점수
%g	|	10진 부동소수점수 혹은 지수로 나타낸 부동소수점수
%%	|	리터럴 %


#### 정렬

```
%[정렬기준(-,없음)][전체글자수].[문자길이 또는 소수점 이후 문자길이][변환타입]
```

```python
>>> d = 37
>>> f = 3.14
>>> s = 'Fastcampus'
>>> '%d %f %s' % (d, f, s)
'37 3.140000 Fastcampus'
>>> '%10d %10f %10s' % (d, f, s)
'        37   3.140000 Fastcampus'
>>> '%14d %14f %14s' % (d, f, s)
'            37       3.140000     Fastcampus'
>>> '%-14d %-14f %-14s' % (d, f, s)
'37             3.140000       Fastcampus    '
>>> '%-14.3d %-14.3f %-14.3s' % (d, f, s)
'037            3.140          Fas           '
```


#### 새 스타일 ({}, format)

```
{}.format(변수)
```

```python
# 기본형태
>>> '{} {} {}'.format(d, f, s)
'37 3.14 Fastcampus'

# 각 인자의 순서를 지정
>>> '{1} {2} {0}'.format(d, f, s)
'3.14 Fastcampus 37'

# 각 인자에 이름을 지정
>>> '{d} {f} {s}'.format(d=50, f=1.432, s='WPS')
'50 1.432 WPS'

# 딕셔너리로부터 변수 할당
>>> dict = {'d': d, 'f': f, 's': s}
>>> '{0[d]} {0[f]} {0[s]} {1}'.format(dict, 'WPS')
'37 3.14 Fastcampus WPS'

# 타입 지정자 입력
>>> '{:d} {:f} {:s}'.format(d, f, s)
'37 3.140000 Fastcampus'

# 이름과 타입지정자를 모두 사용
>>> '{digit:d} {float:f} {string:s}'.format(digit=700, float=1.4323, string='Welcome')
'700 1.432300 Welcome'

# 필드길이 10, 우측정렬
>>> '{:10d}'.format(d)
'        37'
>>> '{:>10d}'.format(d)
'        37'

# 필드길이 10, 좌측정렬
>>> '{:<10d}'.format(d)
'37        '

# 필드길이 10, 가운데 정렬
>>> '{:^10d}'.format(d)
'    37    '

# 필드길이 10, 가운데 정렬, 빈 공간은 ~로 채움
>>> '{:~^10d}'.format(d)
'~~~~37~~~~'
```


#### `f` 표현법

```
f'{변수 또는 표현식}'
```

```
# 기본 형태
apple = '사과'
banana = '바나나'
train = '기차'

f'{apple}는 맛있어 맛있으면 {banana} {banana}는 길어 길면 {train}'

# 옵션 지정
f'''{apple:^10}
{banana:^10}
{train:^10}'''
```


## 시퀀스 타입

파이썬에 내장된 시퀀스 타입에는 문자열, 리스트, 튜플이 있다.
문자열은 인용부호(작은따옴표, 큰따옴표)를 사용하며, 리스트는 대괄호[], 튜플은 괄호()를 사용하여 나타낸다.
시퀀스 타입의 객체는 인덱스 연산을 통해 내부 항목에 접근 할 수 있다.



#### 리스트 항목추가(append)

문자열 항목 하나씩 추가할때 씀

#### 리스트 병합(extend, +=)

리스트안에 문자열이 여러게일경우 병합할때 씀

#### 특정 위치에 리스트 항목 추가(insert)

ex) fruits.insert(1,'mango')




#### 특정 위치 리스트 항목 삭제 (del)

파이썬 구문 **del**을 사용  
> del은 리스트 함수가 아닌, 파이썬 구문이므로 `del <리스트>[오프셋]` 형식을 사용한다.

```
>>> del fruits[0]
```

#### 값으로 리스트 항목 삭제 (remove)

```
>>> fruits.remove('mango')
```

#### 리스트 항목 추출 후 삭제 (pop)

```
>>> fruits.pop()
>>> fruits.pop(-3)
```

#### 값으로 리스트 항목 오프셋 찾기 (index)

```
>>> fruits.index('red')
```

#### 존재여부 확인 (in)

```
>>> 'red' in fruits
True
```

#### 값 세기 (count)

```
>>> fruits.append('red')
>>> fruits.append('red')
>>> fruits.count('red')
3
```

#### 정렬하기 (sort, sorted)

- sort는 리스트 자체를 정렬
- sorted는 리스트의 정렬 복사본을 반환

#### 리스트 복사 (copy)

- copy함수
- list함수
- 슬라이스 연산[:]



### 튜플

튜플은 리스트와 비슷하나, 정의 후 내부 항목의 삭제나 수정이 불가능하다.

#### 튜플 언패킹

```python
>>> f1, f2 = fruits
```

값의 교환

#### 형 변환

**tuple**함수를 사용 (튜플 생성에는 사용 불가능)

리스트를 튜플로 변환

#### 튜플을 사용하는 이유

- 리스트보다 적은 메모리 사용
- 정의후에는 변하지 않는 내부 값




## 딕셔너리(dictionary)

Key-Value형태로 항목을 가지는 자료구조.


#### 딕셔너리 생성

```python
>>> empty_dict1 = {}
>>> empty_dict2 = dict()
>>> champion_dict = {
... 'Lux': 'the Lady of Luminosity',
... 'Ahri': 'the Nine-Tailed Fox',
... 'Ezreal': 'the Prodigal Explorer',
... 'Teemo': 'the Swift Scout',
... }
```

#### 형변환

**dict** 함수를 사용, 두 값의 시퀀스(리스트 또는 튜플)을 딕셔너리로 변환 한다.

```python
>>> sample = [[1,2], [3,4], [5,6]]
>>> dict(sample)
{1: 2, 3: 4, 5: 6}
```

#### 항목 찾기/추가/변경 [key]

```
>>> champion_dict['Lux']
'the Lady of Luminosity'
>>> champion_dict['Sona'] = 'Maven of the Strings'
>>> champion_dict['Lux'] = 'Demacia'
```

#### 항목이 없을 경우 기본값을 지정하고 찾기

```
champion_dict.get('Soraka') # 없을경우 None
champion_dict.get('Soraka', 'Healer') # 없을경우 'Healer'문자열
```

#### 결합 (update)

```
>>> item_dict = {
... 'Doran\'s Ring': 400,
... 'Doran\'s Blade': 450,
... 'Doran\'s Shield': 450,
... }
>>> com_dict = {}
>>> com_dict.update(champion_dict)
>>> com_dict.update(item_dict)
>>> com_dict
```

서로 같은 키가 있을 경우, update에 주어진 딕셔너리의 값이 할당된다.


#### 삭제 (del)

```
>>> del com_dict['Doran\'s Blade']
>>> del com_dict['Doran\'s Ring']
>>> del com_dict['Doran\'s Shield']
```

#### 전체 삭제 (clear)

전체 항목을 삭제


#### in으로 키 검색

True/False를 반환한다.


#### 키 또는 값 얻기

**keys()**  
모든 키 얻기

**values()**  
모든 값 얻기

**items()**  
모든 키-값 얻기 (튜플로 반환)

#### 복사

**copy()**  



## 셋(Set)

셋은 키만 있는 딕셔너리와 같으며, 중복된 값이 존재할 수 없다.


#### 셋 생성

```python
>>> empty_set = set()
>>> champions = {'lux', 'ahri', 'ezreal'}
```

#### 형변환

문자열, 리스트, 튜플, 딕셔너리를 셋으로 변환할 수 있으며, 중복된 값이 사라진다.

```
>>> set('ezreal')
{'e', 'z', 'a', 'l', 'r'}
```

```
>>> set(champion_dict)
{'Ahri', 'Lux', 'Ezreal', 'Sona', 'Teemo'}
```
딕셔너리는 키만 남는다.

#### 집합 연산

연산자|설명
---|---
\|	|	합집합(Union)
&	|	교집합(Intersection)
\-	|	차집합(Difference)
^	|	대칭차집합(Exclusive)
<=	|	부분집합(Subset)
<	|	진부분집합(Proper subset)
\>=	|	상위집합(Superset)
\>	|	진상위집합(Proper superset)

```
>>> A = {1,2,3,4,5}
>>> B = {4,5,6,7,8,9}
>>> C = {4,5,6}
>>> A|B
>>> A&B
>>> A-B
>>> B-A
>>> A^B
>>> A <= B
>>> C <= B
>>> C < B
>>> B <= B
>>> B < B
```



















