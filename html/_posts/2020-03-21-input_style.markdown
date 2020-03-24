---
layout: post
title: "[input] reset"
categories: posts
tags: html_css_js
---

# [input] style 설정

## input style기본설정 reset해서 사용하기
- 기본 스타일이을 없애고 따로 설정하기

```
/* input style */
input {background-image: none;}
input[type=button],
input[type=reset],
input[type=text],
input[type=password],
input[type=submit],
input[type=search],
input[type=tel],
input[type=email] {
    -webkit-appearance: none;
    border-radius: 0
}
input[type=search]::-webkit-search-cancel-button,
input[type=search]::-webkit-search-decoration {
    -webkit-appearance: none
}
input:checked[type=checkbox] {
    background-color: #666;
    -webkit-appearance: checkbox
}
```


