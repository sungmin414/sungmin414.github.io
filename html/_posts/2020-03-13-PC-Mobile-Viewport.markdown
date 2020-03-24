---
layout: post
title: "[html] setting"
categories: posts
tags: html_css_js
---

# [HTML] 기본 셋팅

## PC화면을 Mobile화면보기 뷰포트(Viewport)

> 피시화면의 비율을 줄여 모바일 화면에서 보일수 있도록하는 기술

```
<meta name="viewport" content="속성">
```

### width
- viewport의 가로 크기를 조정

### height
- viewport의 세로 크기를 조정

### initial-scale
- 초기 배율을 설정, 기본값은 1

### minimum-scale
- viewport의 최소 배율값, 기본값은 0.25
- viewport의 최대 배율값, 기본값은 1.6

### user-scalable
- 사용자의 확대/축소 기능을 설정, 기본값은 yes

## 반응형 웹 페이지를 제작하려면 viewport를 설정해야함

```
<meta name="viewport"
	content="user-scalable=no,		(화면 확대 금지)
	initial-scale=1.0,				(초기 배율 설정)
	maximum-scale=1.0,				(최대 배율 설정)
	minimum-scale=1.0,				(최소 배율 설정)
	width=device-width"				(디바이스 가로값 설정)
>
```

- 반응형 페이지를 제작하려면 뷰포트와 미디어쿼리가 필요
- 뷰포트는 화면의 비율을 모바일 화면에 맞게 줄여줌
- 미디어쿼리는 화면 크기에 따라 css를 설정

## X-UA-Compatible, IE=edge 호환성보기(익스플로러에서만 해당)

### 호환성 보기는 브라우저의 최신 렌더링 모드로 작동하기 위해 사용

> 이소스를 쓰면 최신 렌더링 모드를 사용함 

```
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```
- 익스플로러 브라우저를 위함
- 현재는 구 익스를 많이 사용하지 않기 떄문에 죽은 소스임
- 이런게 있었구나 하고 넘어가면 된다

## 기본적으로 넣는 meta테그


```
<meta name="author" content="webstoryboy">
<meta name="description" content="메가박스 사이트 따라하면서 배우는 튜토리얼입니다.">
<meta name="keywords" content="메가박스, 유투브, 영화, 최신영화, 영화관, CGV, 롯데시네마, 웹스토리보이, 웹스, 사이트 만들기, 따라하기">
```

- author (누가 만들었는지)
- description (설명)
- keywords (키워드)

## CSS 사용법

- css코딩 작업한 경로로 들어가서 불러오기

```
<link rel="stylesheet" href="assets/css/reset.css">
<link rel="stylesheet" href="assets/css/style.css">
```

## 파비콘

### 파비콘은 favorites + icon을 합친 말로 인터넷 웹 브라우저의 주소창에 표시되는 웹페이지를 대표하는 아이콘

> 사용법

```
<link rel="shortcut icon" href="favicon.ico">
```

- 사이즈는 16x16, 32x32, 48x48, 64x64
- .ico .png .jpg .gif파일을 지원

> 모바일용 아이콘 사용법


```
<link rel="apple-touch-icon-precomposed" href="favicon.png" />
```
- 안드로이드나 아이폰에서 원하는 페이지를 추가할 경우 아이콘이 생김

1. 파비콘은 웹 브라우저 아이콘이다
2. 파비콘은 .ico 파일로 만들어야함
3. .ico 파일은 모든 브라우저를 지원하고 .png파일은 ie6~10에서는 지원안함
4. 모바일 아이콘도 따로 설정 할 수 있음
5. 모바일 아이콘은 해상도별로 설정

```
예시)

	<!-- 파비콘 -->
	<link rel="shortcut icon" href="assets/icons/favicon.ico">
	<link rel="apple-touch-icon-precomposed" href="assets/icons/favicon_72.png" />
	<link rel="apple-touch-icon-precomposed" sizes="96x96" href="assets/icons/favicon_96.png" />
	<link rel="apple-touch-icon-precomposed" sizes="144x144" href="assets/icons/favicon_144.png" />
	<link rel="apple-touch-icon-precomposed" sizes="192x192" href="assets/icons/favicon_192.png" />
```

## 페이스북, 트위터 메타 태그

### 정보공유하기 

```
    <!-- 페이스북 메타 태그 -->
    <meta property="og:title" content="메가박스 사이트 만들기" />
    <meta property="og:url" content="https://github.com/webstoryboy/megabox2019" />
    <meta property="og:image" content="https://webstoryboy.github.io/megabox2019/mega.jpg" />
    <meta property="og:description" content="메가박스 사이트 따라하면서 배우는 튜토리얼입니다." />
   
    <!-- 트위터 메타 태그 -->
    <meta name="twitter:card" content="summary">
    <meta name="twitter:title" content="메가박스 사이트 만들기">
    <meta name="twitter:url" content="https://github.com/webstoryboy/megabox2019/">
    <meta name="twitter:image" content="https://webstoryboy.github.io/megabox2019/mega.jpg">
    <meta name="twitter:description" content="메가박스 사이트 따라하면서 배우는 튜토리얼입니다.">
```

## 시멘틱태그 인식

+ 시멘틱(Semantic)은 '의미론적인' 이라는 뜻으로 HTML5에서 태그가 의미론적으로 파생되어 생긴 태그들임
+ html5shiv는 구 익스에서 시멘틱 태그를 인식시켜주는 라이브러리임

### 새로 생긴 태그들이 구익스에서 인식하지 못할때 사용

```
    <!-- HTLM5shiv ie6~8 -->
    <!--[if lt IE 9]> 
      <script src="assets/js/html5shiv.min.js"></script>
      <script type="text/javascript">
         alert("현재 브라우저는 지원하지 않습니다. 크롬 브라우저를 추천합니다.!");
      </script>
   <![endif]-->
```


## 제이쿼리 사용법

```
<body>
    
    
    <!-- 자바스크립트 라이브러리 -->
    <script src="assets/js/jquery.min_1.12.4.js"></script>
    <script src="assets/js/modernizr-custom.js"></script>
    <script src="assets/js/ie-checker.js"></script>
</body>
```

