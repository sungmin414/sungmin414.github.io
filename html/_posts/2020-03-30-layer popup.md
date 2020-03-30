# layer popup


## layer popup

- 기본적인 layer popup 사용법

```
예) html.file

<!-- layer popup -->
    <div id="layer">
        <img src="img/webstandard1.jpg" alt="웹표준 사이트">
        <a href="#" class="close">Close</a>
    </div>
<!-- //layer popup -->


예) css.file

/* 레이어 팝업 */
#layer{display: none; position: fixed; left: 50px; top: 50px; width: 700px; border: 10px solid #dceff7;
    box-shadow: 3px 3px 10px rgba(0,0,0,0.4);
}
#layer img{width: 100%; display: block;}
#layer .close{position: absolute; right: 20px; top: 20px ; background: #0093bd; padding: 1px 6px; color: #fff;}
#layer .close:hover{text-decoration: underline;}


예) js.file

// 팝업 메뉴
        $(".layer").click(function(e){
            e.preventDefault();
            // $("#layer").css("display","block");
            // $("#layer").show();
            // $("#layer").fadeIn();
            $("#layer").slideDown();
        });
        $("#layer .close").click(function(e){
            e.preventDefault();
            // $("#layer").show();
            // $("#layer").fadeOut();
            $("#layer").slideUp();
        });
```

## 윈도우 팝업

- 팝업을 따로만들어서 불러와서 쓰기(popup.html)

```
예) html.file

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        *{margin:0; padding: 0;}
        img{width:100%;}
    </style>
</head>
<body>
    <div>
        <img src="img/webstandard2.jpg" alt="웹표준2">
    </div>
</body>
</html>



예) js.file

//윈도우 팝업
        $(".window").click(function(e){
            e.preventDefault();
            //window.open("파일명","팝업이름","옵션설정")
            //옵션 : left, top, width, height, status, toolbar, location, menubar, scroollbars, fullscreen
            window.open("popup.html","popup01","width=800,height=590,left=50,top=50,scrollbar=0,toolbar=0,menubar=0");
        });
```


## 라이트 박스

- [라이트박스갤러리](https://sachinchoolur.github.io/lightGallery/)
- git들어가서 파일받기(압축풀고 파일옮기기)
	- css.file(lightgallery.css)
	- js.file(lightgallery.min.js, lightgallery-all.min.js)
	- font.file
