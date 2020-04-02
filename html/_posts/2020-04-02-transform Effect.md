---
layout: post
title: "[transform] Effect"
categories: posts
tags: html_css_js
---

# [transform] Effect

- transform과 transition을 이용하여 효과주기
- [web's_site](https://wsss.tistory.com/)

```
예) html.file

<div class="side1">
	<figure class="front">
		<img src="img/side1.jpg" alt="이미지1">
	</figure>
	<div class="back">
		<i class="fa fa-heart" aria-hidden="true"></i>
	</div>
</div>

<div class="side2">
	<figure class="front">
		<img src="img/side4.jpg" alt="이미지2">
		<figcaption>Hover Effect</figcaption>
	</figure>
	<figure class="back">
		<img src="img/side2.jpg" alt="이미지2">
		<figcaption>Hover Effect</figcaption>
	</figure>
</div>

<div class="side3">
	<figure>
		<img src="img/side3.jpg" alt="이미지3">
		<figcaption><h3>Hover<em>Effect</em></h3></figcaption>
	</figure>
</div>

-----------------------
예) css.file

/*사이트 이펙트1*/
.side1 {
    position: relative;
    display: block;
    perspective: 600px;
}

.side1 .front {
    transform-style: preserve-3d;
    transform: rotateY(0deg);
    transition: all 0.5s ease-in-out;
    backface-visibility: hidden;
}

.side1 .back {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1;
    transform-style: preserve-3d;
    color: #fff;
    background: #4038dc;
    transform: rotateY(-180deg);
    transition: all 0.5s ease-in-out;
    backface-visibility: hidden;
}

.side1:hover .front {
    transform: rotateY(180deg);
}

.side1:hover .back {
    transform: rotateY(0deg);
    z-index: 1;
}
.side1 .back i{position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%); font-size: 60px;}


/*사이드 이펙트2*/
.side2 {position: relative; display: block; perspective: 600px;}
.side2 .front{
	transform-style: preserve-3d;
	transform: rotateY(0deg);
	transition: all 0.5s ease-in-out;
	backface-visibility: hidden;
}
.side2 .back{
	position: absolute; top: 0; left: 0;
	width: 100%; height: 100%; z-index: -1;
	transform-style: preserve-3d;
	transform: rotateY(180deg);
	transition: all 0.5s ease-in-out;
	backface-visibility: hidden;
}
.side2 .front figcaption{
	position: absolute; left: 50%; top: 50%;
	transform: translate(-50%, -50%) translateZ(100px);
	display: block;
	text-align: center;
}
.side2 .back figcaption{
	position: absolute; left: 50%; top: 50%;
	transform: translate(-50%, -50%) translateZ(100px);
	display: block;
	text-align: center;
}
.side2 figcaption{width: 60%; color: #fff; font-size: 20px; font-family: 'Abel'; font-weight: bold; background: rgba(0,0,0,0.4); padding: 3px 10px;}

.side2:hover .front{transform: rotateY(180deg);}
.side2:hover .back{transform: rotateY(0deg); z-index: 1;}

/*사이드 이팩트3*/
.side3{position: relative; overflow: hidden; }
.side3 figcaption{
	position: absolute; top: 50%; left: 50%;
	color: #fff; text-align: center;
	opacity: 0;
	text-transform: uppercase;
	transition: all 0.3s ease;
	transform: translate(350%, -50%) rotate(180deg);

}
.side3 figcaption:after{
	content: '';
	width: 100px; height: 100px;
	background: #000; border-radius: 50%;
	position: absolute; left: 50%; top: 50%; z-index: -1;
	transform: translate(-50%, -50%);
}
.side3 figcaption h3{font-size: 16px;}
.side3 figcaption em{display: block; font-weight: bold;}
.side3 img{display: block; transition: all 0.3s ease;}
.side3:hover img{opacity: 0.4;}

.side3:hover figcaption{
	transform: translate(-50%, -50%) rotate(0deg); opacity: 1;	
}

```
