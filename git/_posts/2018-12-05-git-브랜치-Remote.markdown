---
layout: post
title: "[Git] 브랜치-Remote"
categories: posts
tags: git
---

# [Git]브랜치-Remote


## Remote 브랜치란?

리모트 브랜치란 리모트 저장소에 있는 브랜치를 말한다. 
사실 리모트 브랜치도 로컬에 있지만 멋대로 옮기거나 할 수 없고 리모트 저장소와 통신하면 자동으로 업데이트된다. 
리모트 브랜치는 브랜치 상태를 알려주는 책갈피라고 볼 수 있다. 
이 책갈피로 리모트 저장소에서 마지막으로 데이터를 가져온 시점의 상태를 알 수 있다.


**리모트 서버로부터 저장소 정보를 동기화**

```
동기화
git fetch origin

브랜치 합치기
git merge origin/marster
```


