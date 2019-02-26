---
layout: post
title: "[Git] 브랜치-Remote"
categories: posts
tags: git
---

# [Git]브랜치-Remote, Rebase


## Remote 브랜치란?

리모트 브랜치란 리모트 저장소에 있는 브랜치를 말한다. 
사실 리모트 브랜치도 로컬에 있지만 멋대로 옮기거나 할 수 없고 리모트 저장소와 통신하면 자동으로 업데이트된다. 
리모트 브랜치는 브랜치 상태를 알려주는 책갈피라고 볼 수 있다. 
이 책갈피로 리모트 저장소에서 마지막으로 데이터를 가져온 시점의 상태를 알 수 있다.


**리모트 서버로부터 저장소 정보를 동기화**

+ 리모트 브랜치는 origin/master 로 표기됨.

```
동기화
git fetch origin

브랜치 합치기
git merge origin/marster
```


## Rebase 하기

+ Merge와 결과는 같음
+ Merge를 사용하면 일반적으로 브랜치를 나눠서 작업을하면 log에 가지를쳐서 커밋내용을 남기는데 
Rabase를 사용하면 log를 보면 가지를치는게 아니고 일직선으로 쭉 나열됨
+ 히스토리를 정리하기 위해서는 Rebase를 사용할수도 있지만 리모트 등 어딘가에 Push로 보낸 커밋에 대해서는
절대 Rebase 하지 말아야함


```
최신 소스 받기 (fetch, rebase 한번에하는 )
git pull --rebase upstream master
```


## commit 되돌리기

```
커밋 되돌리기 마지막 숫자는 몇개 되돌릴것인지 지
git reset HEAD~1
```

## Stash란?

+ Stash는 임시저장소이다
+ modified 파일 즉 스냅샷 되기전 파일이 있을때 브랜치 이동이 안됨

```
modified 파일을 임시저장소에 저장
git stash

임시저장소에 있는걸 다시 사용하겠다는 명령어
git stash pop
```