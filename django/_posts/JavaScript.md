# JavaScript, Jquery

### HTML5shiv, jquery 사용법

```
# /head 위에 넣기

	<!-- HTLM5shiv ie6~8 -->
    <!--[if lt IE 9]> 
        <script src="js/html5shiv.min.js"></script>
        <script type="text/javascript">
            alert("현재 당신이 보는 브라우저는 지원하지 않습니다. 최신 브라우저로 업데이트해주세요!");
        </script>
    <![endif]-->
    
    
# /body 위에 넣기

    <!-- JavaScript Libraries -->
    <script src="js/jquery.min_1.12.4.js"></script>
    <script src="js/modernizr-custom.js"></script>
```


### 버튼 슬라이드효과

+ 버튼을 클릭했을때 슬라이드효과 종류
	+ show
	+ fadeIn
	+ slideDown
	+ toggle
	+ fadeToggle

```
예)

<script type="text/javascript">
//        버튼을 클릭하면 전체 메뉴를 보이게 해줌
        $(".title .btn").click(function(e){
//            e.preventDefault(); # 기능을 깨줌
            e.preventDefault();
//            $("#cont_nav").css("display","block");
//            $("#cont_nav").show();
//            $("#cont_nav").fadeIn();
//            $("#cont_nav").slideDown();
//            $("#cont_nav").toggle();
//            $("#cont_nav").fadeToggle();
            $("#cont_nav").slideToggle(200);
            $(this).toggleClass("on");
        });
    </script>
    
```


