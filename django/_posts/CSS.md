# CSS

+ [html, css, Javascript 통계사이트](https://caniuse.com)
	+ html, css, Javascript 현재 웹브라우저들이 얼마나 채택하고있는지 보여주는 사이트
+ [CSS 문서](http://tcpschool.com/css/intro)


### 스킵 내비게이션

+ html파일

```
예)

<div id="skip">
    <a href="#cont_nav">전체 메뉴 바로가기</a>
    <a href="#cont_banner">배너 영역 바로가기</a>
    <a href="#cont_content">컨텐츠 영역 바로가기</a>
</div>
```

+ style.css

```
/*스킵내비게이션*/

#skip{position: relative;}
#skip a{position: absolute; left: 0; top: -35px; border: 1px solid #fff; 
			color: #fff; background: #333; line-height: 30px; 
			width: 140px;  text-align: center;}
#skip a:active,
#skip a:focus{top: 0;}
```

### Css 

> [CSS 정리 문서](https://webzz.tistory.com/353)

+ letter-spacing: 0px;
	+ 글자 간격
+ visibility: hidden;
	+ 요소가 차지하던 공간은 그대로 유지되고, 화면에 보이지만 않게 된다.
+ transition: color(속성) 0.3s ease(효과);
	+ 시간차 주기, 요소의 움직임 


#### 텍스트 한 줄 효과
+ overflow: hidden;
	+ 영역에서 벗어난 것은 안보이게함
+ text-overflow: ellipsis;
	+ 내가 지정한영역을 넘어가면 ...으로 나옴
+ white-space: nowrap;
	+ 두줄을 한줄로 바꿔줌 

#### 텍스트 두 줄 효과

+ overflow, text-overflow 위와동일

```
# box는 개발단계라 -webkit를 붙여서 사용

display: -webkit-box;
	-webkit-box-orient: vertical;
	-webkit-line-clamp: 2;      <- 숫자에 따라 효과 바뀜
```



### grid
+ grid
+ 사용법 
	+ [grid 사용법 문서](https://www.w3schools.com/cssref/pr_grid.asp)

```
# 사용법( 열과행을 지정할수 있음, 자세한건 검색해서 사용하기 )

display: grid;
grid-template-columns: 150px 1fr;
```

### 미디어 쿼리

+ [media 사용법 문서](https://www.w3schools.com/cssref/css3_pr_mediaquery.asp)

> 반응형 layout 화면 size

+ 화면 너비가 0 ~ 1280px : 데스크탑
+ 화면 너비가 0 ~ 1024px : 데스크탑
+ 화면 너비가 0 ~ 960px : 노트북
+ 화면 너비가 0 ~ 768px : 테블릿
+ 화면 너비가 0 ~ 576px : 모바일

> 미디어 쿼리는 화면 크기에 따른 각각의 속성 값을 지정하여, 여러가지 화면을 구성하는 기술

+ 사용법
	+ @media only all and (조건문) {실행문}
	
```
@media : 미디어 쿼리가 시작됨을 표시
only: 미디어 쿼리 구문을 해석하라는 명령어(생략가능)
all: 미디어 쿼리를 해석해야 할 대상을 나타냄(생략가능)
	all : 모든 미디어 유형에서 사용할 CSS를 정의
	print : 인쇄 장치에서 사용할 CSS를 정의
	screen : 컴퓨터 스크린에서 사용할 CSS를 정의
	aural : 화면을 읽어 소리로 출력해주는 장치에서 CSS를 정의
	tv : TV에서 사용할 CSS를 정의
	handheld : 손에 들고 다니는 장치를 사용힐 CSS를 정의
	projection : 프로젝트를 위한 사용할 CSS를 정의
and : 앞과 뒤의 조건을 나타냄(생략가능)
(조건문) : 해당 조건을 설정할 수 있음
{실행문} : 조건에 따른 실행을 설정
```

### 이미지 해상도 속성 보기
+ [이미지 해상도 속성값 보여주는 사이트](https://webkit.org/demos/srcset/)
+ [고밀도 디스플레이를 위한 이미지 다루는법 정리 된 사이트](https://blog.hanlee.io/2018/high-density-display-and-images/)

### lightgallery 플러그인

+ [lightgallery플로그인 사용법 문서](https://sachinchoolur.github.io/lightGallery/)


