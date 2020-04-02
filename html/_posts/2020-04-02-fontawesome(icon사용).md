---
layout: post
title: "[fontawesome] icon사용"
categories: posts
tags: html_css_js
---


# [fontawesome] icon사용

**[fontawesom_site(icon)](https://fontawesome.com/)**

- 프로젝트 파일안에 css, font파일 넣어주기
- html.file안에 link설정 해주기
- fontawesome에서 사용할 아이콘 찾아서 사용하기



## border 응용

- border값으로 화살표 만들기

```
예) html.file

<div class="header_icon">
	<ul>
		<li><a href="#"><i class="fa fa-html5" aria-hidden="true"></i><span>HTML5</span></a></li>
		<li><a href="#"><i class="fa fa-github" aria-hidden="true"></i><span>Github</span></a></li>
		<li><a href="#"><i class="fa fa-facebook-square" aria-hidden="true"></i><span>Facebook</span></a></li>
		<li><a href="#"><i class="fa fa-twitter" aria-hidden="true"></i><span>twitter</span></a></li>
	</ul>
</div>
<!-- //header-icon -->


-------------------------------------------------------
예) css.file

.header .header_icon{
	text-align: center; margin-top: 40px; padding-bottom: 45px;
}
.header .header_icon li{display: inline; margin: 0 2px;}
.header .header_icon li a{
	position: relative;
	background-color: #3192bf;
	border-radius: 50%;
	width: 60px; height: 60px; color: #fff;
	display: inline-block;
	font-size: 35px;
	line-height: 60px;
	transition: all 0.3s ease;
}
.header .header_icon li a span{
	position: absolute;
	left: 50%; top: -40px;
	font-size: 12px;
	line-height: 1.6;
	background: #3192bf;
	transform: translate(-50%);
	padding: 3px 9px;
	border-radius: 6px 0;
	opacity: 0;
	transition: all 0.3s ease;
}

.header .header_icon li a span:before{
	content: '';
	position: absolute;
	left: 50%; bottom: -5px;
	margin-left: -5px;
	border-top: 5px solid #3192bf;
	border-left: 5px solid transparent;
	border-right: 5px solid transparent;
}
.header .header_icon li a:hover span{
	opacity: 1;
	top: -33px;
}

.header .header_icon li a:hover{
	box-shadow: 
		0 0 0 3px rgba(75,154,191,0.9) inset,
		0 0 0 100px rgba(0,0,0,0.1) inset;
}
```

