# HTML(layout, tag, 댓글, 채팅, 방문자)

## layout

+ media쿼리 반응형 레이아웃 

```
# layout.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="author" content="webstoryboy">
    <meta name="description" content="반응형 사이트 따라하기">
    <meta name="keywords" content="반응형사이트, 웹퍼블리셔, 웹접근성, HTML5, webstoryboy, webs">
    <title>Document</title>
    
	<!-- HTLM5shiv ie6~8 -->
    <!--[if lt IE 9]> 
        <script src="js/html5shiv.min.js"></script>
        <script type="text/javascript">
            alert("현재 당신이 보는 브라우저는 지원하지 않습니다. 최신 브라우저로 업데이트해주세요!");
        </script>
    <![endif]-->
    
</head>
<body>
    <div id="wrap">
        <header id="header"></header>
        <section id="banner"></section>
        <section id="content1">
            <nav class="nav"></nav>
            <article class="article_right1"></article>
            <article class="article_right2"></article>
        </section>
        <section id="content2">
            <article class="article_box1"></article>
            <article class="article_box2"></article>
            <article class="article_box3"></article>
        </section>
        <section id="content3">
            <article class="article_square1"></article>
            <article class="article_square2"></article>
            <article class="article_square3"></article>
            <article class="article_square4"></article>
        </section>
        <footer id="footer"></footer>
    </div>
    
    
    <!-- JavaScript Libraries -->
    <script src="js/jquery.min_1.12.4.js"></script>
    <script src="js/modernizr-custom.js"></script>
</body>
</html>
```

```
# style.css

* {margin: 0; padding: 0; }

#wrap { width: 1200px; margin: 0 auto; }
#header { width: 100%; height: 100px; background-color: aqua; }
#banner { width: 100%; height: 300px; background-color: aquamarine; }

#content1 { overflow: hidden; }
#content1 .nav { float: left; width: 30%; height: 300px; background-color: quewhite; }
#content1 .article_right1 { float: left; width: 70%; height: 150px; background-color: mediumaquamarine; }
#content1 .article_right2 { float: left; width: 70%; height: 150px; background-color: black; }

#content2 { overflow: hidden; } 
#content2 .article_box1 { float: left; width: 33.3333%; height: 300px; background-color: darkturquoise; }
#content2 .article_box2 { float: left; width: 33.3333%; height: 300px; background-color: paleturquoise; }
#content2 .article_box3 { float: left; width: 33.3333%; height: 300px; background-color: turquoise; }

#content3 { overflow: hidden; background-color: aqua; padding: 2%}
#content3 > article { margin: 1%; border-radius: 16px}
#content3 .article_square1 { float: left; width: 23%; height: 200px; background-color: magenta; }
#content3 .article_square2 { float: left; width: 23%; height: 200px; background-color: maroon; }
#content3 .article_square3 { float: left; width: 23%; height: 200px; background-color: mediumaquamarine; }
#content3 .article_square4 { float: left; width: 23%; height: 200px; background-color: moccasin; }

#footer { width: 100%; height: 100px; background-color: paleturquoise; }

/*        화면너비 1220px */
@media(max-width:1220px) {
    #wrap { width: 100%; }

    #content2 .article_box1 { width: 50% }
    #content2 .article_box2 { width: 50% }
    #content2 .article_box3 { display: none }

    #content3 .article_square1 { width: 31.3333%; }
    #content3 .article_square2 { width: 31.3333%; }
    #content3 .article_square3 { width: 31.3333%; }
    #content3 .article_square4 { display: none; }
}

/*        화면너비 768px */
@media(max-width:768px) {
    #content2 .article_box1 { width: 100%; height: 150px; }
    #content2 .article_box2 { width: 100%; height: 150px; }
}

/*        화면너비 500px */
@media(max-width:500px) {
	 #content1 .nav { width: 100%; height: 150px; }
    #content1 .article_right1 { width: 100%; height: 150px; }
    #content1 .article_right2 { width: 100%; height: 150px; }

    #content3 .article_square1 { width: 98%; height: 150px; }
    #content3 .article_square2 { width: 98%; height: 150px; }
    #content3 .article_square3 { width: 98%; height: 150px; }
}
```


## 명령어

+ [html 많이사용하는 테그 통계 사이트](https://advancedwebranking.com/html/)

### tag

+ [html 문서 사이트](http://tcpschool.com/html/intro)




### table 만들기

```
# table 사용법

<table border="1">
    <tr>
        <td>head</td>
        <td>98.1%</td>
    </tr>
    <tr>
        <td>body</td>
        <td>97.9%</td>
    </tr>
    <tr>
        <td>html</td>
        <td>97.9%</td>
    </tr>
    
</table>
```

| 태그 | 비고 |
|---|---|
| `<table>` | 테이블을 만드는 태그 |
| `<th>` | 테이블의 헤더부분을 만드는 태그 |
| `<tr>` | 테이블의 행을 만드는 태그 |
| `<td>` | 테이블의 열을 만드는 태그 |



## 댓글기능 서비스

+ [disqus사이트](https://disqus.com)

+ [livere사이트](https://www.livere.com)

## 채팅기능 서비스

+ [무료 채팅기능 서비스 사이트](https://www.tawk.to)

```
설정 -> widget code 복사해서 사용
```


## 페이지 분석기

+ [google 페이지 분석기](https://analytics.google.com/analytics/web/provision/?authuser=0#/provision)
