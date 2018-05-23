# bit Crop [![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

「 图片 裁剪 `crop` 简单 说明 」

Explanation

> "version": "1.0.0"

[github source](https://github.com/netplayer/crop)

[中文](./readme.md) | ~~[english](./readme.en.md)~~

[DEMO](http://netplayer.gr/crop/)

---

本次项目更多的是, 解释如何在浏览器上 裁剪图片

如何工作的, 示例

---

本目录

---

### index.html

``` html
      <script type='text/javascript' src='http://code.jquery.com/jquery-latest.min.js'></script>
      <script type='text/javascript' src='crop.js'></script>
```

需要 `jquery` 和 需要说明的 `crop.js`

### crop.js

``` js
$(document).ready(function() {

    var condition = 1;
    var points = [];//holds the mousedown points
    var canvas = document.getElementById('myCanvas');
    this.isOldIE = (window.G_vmlCanvasManager); // 是不是 旧的 IE

```

既然需要裁剪图片, 自然需要图片

``` html
<!-- index.html -->
  <canvas width="217" height="275" id="myCanvas" style="position:relative;margin-left:0px;margin-top:0px;"></canvas>
```

这是 [canvas](https://developer.mozilla.org/zh-CN/docs/Glossary/Canvas) 

你可以理解为 `canvas == 画布`

现在是空白的画布

#### init

``` js
    $(function() {
      //  if (document.domain == 'localhost') {

            if (this.isOldIE) {
                G_vmlCanvasManager.initElement(myCanvas);
            }
            var ctx = canvas.getContext('2d'); // 从画布中拿到画笔
            var imageObj = new Image(); // 新建一张照片



            function init() {
                canvas.addEventListener('mousedown', mouseDown, false);
                canvas.addEventListener('mouseup', mouseUp, false);
                canvas.addEventListener('mousemove', mouseMove, false);
            } // 准备初始化 函数

            // 准备 照片element 获取完照片后
            imageObj.onload = function() {
                // 将 图片 画到 画布上
                ctx.drawImage(imageObj, 0, 0);

            };
            imageObj.src = "img.png";


```

### 定义裁剪区块

[example-for-globalCompositeOperation w3cschool](http://www.w3school.com.cn/tags/canvas_globalcompositeoperation.asp)

1. 保存在图片点击的坐标

2. 显示给用户看 裁剪区块

``` js


            ctx.globalCompositeOperation = 'destination-over'; // 在源图像上方显示目标图像

            // 指针设备( 通常指鼠标 )在元素上移动时, mousemove 事件被触发。
            $('#myCanvas').mousemove(function(e) { 
                if (condition == 1) {

                    // 起始一条路径，或重置当前路径
                    ctx.beginPath();

                    // 每时每刻在图片上的 鼠标坐标
                    $('#posx').html(e.offsetX);
                    $('#posy').html(e.offsetY);
                }
            });
            // 指针设备( 通常指鼠标 )在元素上点击时, mousedown 事件被触发。
            $('#myCanvas').mousedown(function(e) {
                if (condition == 1) {

                    if (e.which == 1) {
                        var pointer = $('<span class="spot">').css({ // 2. 定义好 裁剪点
                            'position': 'absolute',
                            'background-color': '#000000',
                            'width': '5px',
                            'height': '5px',
                            'top': e.pageY,
                            'left': e.pageX


                        });
                        
                        points.push(e.pageX, e.pageY); // 1. 

                        ctx.globalCompositeOperation = 'destination-out'; 
                        // 在源图像外显示目标图像。只有源图像外的目标图像部分会被显示，源图像是透明的。

                        var oldposx = $('#oldposx').html();
                        var oldposy = $('#oldposy').html();
                        var posx = $('#posx').html();
                        var posy = $('#posy').html();

                        // 2. 裁剪 线
                        ctx.beginPath();
                        ctx.moveTo(oldposx, oldposy);
                        if (oldposx != '') {
                            ctx.lineTo(posx, posy);

                            ctx.stroke();
                        }
                        // 2. 裁剪 线

                        // 记录一下
                        $('#oldposx').html(e.offsetX);
                        $('#oldposy').html(e.offsetY);
                    }
                    $(document.body).append(pointer); // 2. 写入 网页, 展示 裁剪点
                    $('#posx').html(e.offsetX);
                    $('#posy').html(e.offsetY);
                }//condition
            });

```

#### 裁剪

1. 移除 那些裁剪区块

2. 清理 画布

``` js
            $('#crop').click(function() {
                condition = 0;

                $('.spot').each(function() { // 1.
                    $(this).remove();

                })

                //  在给定的矩形内清除指定的像素
                ctx.clearRect(0, 0, 217, 275);
                ctx.beginPath();
                ctx.width = 217;
                ctx.height = 275;
                ctx.globalCompositeOperation = 'destination-over'; // 在源图像上方显示目标图像
                // 2. ⬆️

                setTimeout(function() {


                    var offset = $('#myCanvas').offset();


                    for (var i = 0; i < points.length; i += 2) {
                        var x = parseInt(jQuery.trim(points[i]));
                        var y = parseInt(jQuery.trim(points[i + 1]));


                        if (i == 0) {
                            // 把路径移动到画布中的指定点，不创建线条
                            ctx.moveTo(x - offset.left, y - offset.top);
                        } else {
                            // 添加一个新点，然后在画布中创建从该点到最后指定点的线条
                            ctx.lineTo(x - offset.left, y - offset.top);
                        }
                    }

                    if (this.isOldIE) { // 旧

                        ctx.fillStyle = '';
                        ctx.fill();
                        var fill = $('fill', myCanvas).get(0);
                        fill.color = '';
                        fill.src = element.src;
                        fill.type = 'tile';
                        fill.alignShape = false;
                    }
                    else {
                        // 在指定的方向上重复指定的元素
                        var pattern = ctx.createPattern(imageObj, "repeat");
                        ctx.fillStyle = pattern;
                        ctx.fill(); // 填充

                        var dataurl = canvas.toDataURL("image/png"); // 变数据链接
                        
                        // 为什么成功裁剪, 
                        // 0. 清理 画布
                        // 1. 将 裁剪区块 放上去

                        // 2. 将 画 放上去 

                        // 3. 填充画, 就只出现在 区块内能看到了
                        

                        // 这个时候, 其实已经概括好 裁剪的内容
                        // 作者加了个 上传的小示范, 可以看完它
                        var xhr = new XMLHttpRequest();
                        // // 
                        xhr.open('POST', 'upload.php', false);
                        xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded"); // 请求头
                        var files = dataurl;
                        var data = new FormData(); // post:from 格式
                        var myprod = $("#pid").val(); // 这行没什么用
                        data = 'image=' + files;
                        xhr.send(data); // 带入 post from 传送,这样服务器也能查看到变化的图片
                        if (xhr.status === 200) {
                            console.log(xhr.responseText); // 拿到编号🆔
                            $('#myimg').html('<img src="upload/' + xhr.responseText + '.png"/>');
                        }



                    }
                }, 20);

            });

       // }
    });

});

```

#### upload

因为是`PHP` 我简单说一下

``` php
<?php
if($_POST) { // 是不是 post 方式
	
    define('UPLOAD_DIR', 'upload/'); // define() 函数定义一个常量 约等于 js:const
    $img = $_POST['image']; // 拿到 from

    // 图片数据链接 === 
    // data:image/png;base64, 一长串sdfjasldfj

    $img = str_replace('data:image/png;base64,', '', $img);  // 去掉前缀
    //
   
    $img = str_replace(' ', '+', $img); // 有空格就➕
   
    $dataimg = base64_decode($img); // 解码
    
    $nameimg= uniqid() ; // 拿到 随机iD

    $fileimg = UPLOAD_DIR . $nameimg . '.png'; // 图片链接
    $successimg = file_put_contents($fileimg, $dataimg); // file_put_contents() 函数把一个字符串写入文件中。

     echo $nameimg; // echo 就是 本次 post 的返回 responseText， 这样
}
?>

```

$nameimg 拿到ID

> 也就是 `upload/' + xhr.responseText + '.png` 那里 拿到图片

``` html
<!-- nameimg === xhr.responseText -->
$('#myimg').html('<img src="upload/' + xhr.responseText + '.png"/>');

```


### 如何 只拿 裁剪区域外的 图片

1. 不 清理 画布

2. `ctx.globalCompositeOperation = 'xor'` 使用异或操作对源图像与目标图像进行组合

3. `ctx.fill()` 填充

> 尝试 http://llever.com/crop-simple-explain

[剪切版 - crop](./crop.js)

``` js
            $('#crop').click(function() {
                condition = 0;

                //  var pattern = ctx.createPattern(imageObj, "repeat");
                //ctx.fillStyle = pattern;
                $('.spot').each(function() {
                    $(this).remove();

                })

                //  1. 不清理 画布

                ctx.globalCompositeOperation = 'xor'; // 2. 
                //draw the polygon
                setTimeout(function() {


                    //console.log(points);
                    var offset = $('#myCanvas').offset();


                    for (var i = 0; i < points.length; i += 2) {
                        var x = parseInt(jQuery.trim(points[i]));
                        var y = parseInt(jQuery.trim(points[i + 1]));


                        if (i == 0) {
                            ctx.moveTo(x - offset.left, y - offset.top);
                        } else {
                            ctx.lineTo(x - offset.left, y - offset.top);
                        }
                    }
                        ctx.fill() // 3.

                    // }
                }, 20);

            });

       // }
    });

});


```