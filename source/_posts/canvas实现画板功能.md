---
title: canvas实现画板功能
date: 2016-08-11 02:29:37
tags: canvas
categories: Html5
keywords: canvas h5 画板
description: canvas实现画板功能
---
canvas实现画板功能
======
>需求分析
>1.具有基本的铅笔功能
>2.即时直线绘制
>3.即时绘制矩形
>4.即时圆形
>5.即时橡皮擦
>7.插入图片
>8.把画板保存为图片并下载  //此处还没做好，画板底色是黑色的
>9.选择颜色
>10.选择线条粗细


Setp1:构建网页基本样式
-----
>html代码
```html
    <!--可选操作-->
    <div id="select">
        <button id="pen">铅笔</button>
        <button id="line">直线</button>
        <button id="rect">矩形</button>
        <button id="arc">圆形</button>
        <button id="robber">橡皮檫</button>
        <button id="img">图片</button>
        <button id="save">保存</button>
        <input type="file" id="file" name="img" style="display: none"/>
        <input type="color" id="color"/>
        <input type="number" id="lineWidth"/>
    </div>

    <!--画板-->
    <canvas id="penal" width="800" height="800"></canvas>
```

>css代码
```css
    /`只有一句，用来初始化画板颜色`/
    #penal {
        width: 800px;
        height: 800px;
        background-color: #ccc;
    }
```
Step2:构造实现画板各个功能的js代码
---
>1.构造`Draw()`类，并初始化必要的参数
```js
    var Draw = function(){
        this.type = 'pen';  //选项类型，默认为铅笔
        this.penal = document.getElementById('penal');
        this.pen = this.penal.getContext('2d');
        this.isDraw = false;    //绘画开关
        this.color = document.getElementById('color');
        this.lineWidth = document.getElementById('lineWidth');
        this.select = document.getElementById('select');    //选择面板
        this.img = new Image();//用于动态绘制直线，矩形，圆形
    };
```
>2.这里只需要构造一个初始化函数`init()`

>>初始化函数内部需要用到的变量
```js
    Draw.prototype.init = function(){
        var self = this;
        var originX = null;
        var originY = null;
    }
````

>>先绑定选择操作
```js
    this.select.addEventListener('click',function(event){
        if(event.target.id == 'pen'){
            self.type = 'pen';
        }else if(event.target.id == 'line'){
            self.type = 'line';
        }else if(event.target.id == 'rect'){
            self.type = 'rect';
        }else if(event.target.id == 'arc'){
            self.type = 'arc';
        }else if(event.target.id == 'robber'){
            self.type = 'robber';
        }else if(event.target.id == 'img'){
            document.getElementById('file').click();    //默认触发选择文件操作
            document.getElementById('file').onchange = function(e){
                var reader = new FileReader();          //这是H5新增加的读取文件函数
                reader.readAsDataURL(e.target.files[0]);
                reader.onload = function(ent){          //文件读取完毕后触发操作
                    var img = new Image();
                    img.src = ent.target.result;        //读取的结果默认存放在result上
                    self.pen.drawImage(img,0,0);        //把图片直接画在画布上
                }
            }
        }
        else if(event.target.id == 'save'){
            var a = document.createElement('a');
            a.href = self.penal.toDataURL('image/png');     //把画布转化为base64
            a.download = 'image.jpeg';
            a.id = 'download';
            a.innerHTML = 'download';
            document.body.appendChild(a);
            document.getElementById('download').style.display = 'none';
            document.getElementById('download').click();    //默认出发下载操作

        }
    },false);
```

>>可以知道当绘画时，`mousedown`事件触发时，需要把`this.isDraw = true`,同时记录鼠标所在坐标，获取选择的`color`和`linewidth`,并开启绘画路径
```js
    this.penal.addEventListener('mousedown',function(event){
        self.isDraw = true;
        originX = event.clientX - self.penal.offsetLeft;    //原点x坐标
        originY = event.clientY - self.penal.offsetTop;     //原点y坐标
        self.pen.moveTo(originX, originY);
        self.pen.strokeStyle = self.color.value;
        self.pen.lineWidth = self.lineWidth.value;
        self.pen.beginPath();

    },false);
```
>>然后当触发`mouseup`事件时，可知需要结束绘画，若鼠标离开画布，即`mouseleave`时，也需要结束绘画，这部分很简单只需把`this.isDraw = false`和关闭绘画路径
```js
    this.penal.addEventListener('mouseleave', function () {
        if(self.isDraw){
            self.pen.closePath();
            self.isDraw = false;
        }
    },false);
    this.penal.addEventListener('mouseup', function (event) {
        if(self.isDraw){
            self.pen.closePath();
            self.isDraw = false;
        }
    },false);
```
>>接下来就是最难的部分的，当`mouseover`时，需要即时绘制
>>>先实现铅笔功能
```js
    this.penal.addEventListener('mousemove',function(event){
        //只有可绘画时才可画
        if(self.isDraw){
            var x = event.clientX - self.penal.offsetLeft;  //移动过程中的x坐标
            var y = event.clientY - self.penal.offsetTop;   //移动过程中的y坐标

            if(self.type == 'pen'){
                self.pen.lineTo(x,y);
                self.pen.stroke();
            }
        }
    },false);
```
>>>接下来实现橡皮擦功能，实现方法是把绘画线条加粗，并颜色默认选择画布底色
```js
    if(self.type == 'robber'){
         self.pen.strokeStyle = '#ccc';
         self.pen.clearRect(x-10,y-10,20,20);
    }
```
>>>接下来实现绘制直线，矩形和圆形的方法其实大同小异,然后为了能让我们画矩形和圆形能在所有方向都能画，我们增加了`newOriginX`和`newOriginY`两个变量
```js
    var newOriginX  = originX,newOriginY = originY;

    if(self.type == 'line'){
        self.pen.moveTo(originX,originY);
        self.pen.lineTo(x,y);
        self.pen.stroke();

    }else if(self.type == 'rect'){
        if(x < originX){
            newOriginX = x;
        }
        if(y < originY){
            newOriginY = y;
        }
        self.pen.rect(newOriginX,newOriginY,Math.abs(x-originX),Math.abs(y-originY));
        self.pen.stroke();
    }else if(self.type == 'arc'){
        if(x < originX){
            newOriginX = x;
        }
        if(y < originY){
            newOriginY = y;
        }
        var r = Math.sqrt(Math.abs(x-originX) * Math.abs(x-originX) + Math.abs(y-originY) * Math.abs(y-originY))
        self.pen.arc(Math.abs(x-originX)+newOriginX, Math.abs(y-originY)+newOriginY , r, 0, 2*Math.PI);
        self.pen.fillStyle = self.color.value;
        self.pen.fill();
    }
```
>>>`question:`此时我们发现画的直线，矩形和圆形都会在画的时候留下移动的痕迹，这不是我们希望的结果，所以解决方法是：
>>>在`mousedown`时，把当前画布内容保存为图片，并用初始化时一直没有使用过的`this.img`来保存，然后每次画直线等的时候先把画布全部清空，然后在把`this.img`画到画布上
>>>把原来的`mousedown`事件添加一句代码
```js
    this.penal.addEventListener('mousedown',function(event){
        self.isDraw = true;

        //增加一句代码
        self.img.src = self.penal.toDataURL('image/png');

        originX = event.clientX - self.penal.offsetLeft;    //原点x坐标
        originY = event.clientY - self.penal.offsetTop;     //原点y坐标
        self.pen.moveTo(originX, originY);
        self.pen.strokeStyle = self.color.value;
        self.pen.lineWidth = self.lineWidth.value;
        self.pen.beginPath();

    },false);
```
>>>把`mouseover`事件的代码稍作更改
```js
    if(self.type == 'line'){

        self.pen.clearRect(0,0,800,800);//增加代码
        self.pen.drawImage(self.img, 0, 0);//增加代码
        self.pen.beginPath();//增加代码

        self.pen.moveTo(originX,originY);
        self.pen.lineTo(x,y);
        self.pen.stroke();

        self.pen.closePath();//增加代码
    }else if(self.type == 'rect'){
        self.pen.clearRect(0,0,800,800);//增加代码
        self.pen.drawImage(self.img, 0, 0);//增加代码
        self.pen.beginPath();//增加代码

        if(x < originX){
            newOriginX = x;
        }
        if(y < originY){
            newOriginY = y;
        }
        self.pen.rect(newOriginX,newOriginY,Math.abs(x-originX),Math.abs(y-originY));
        self.pen.stroke();

        self.pen.closePath();//增加代码
    }else if(self.type == 'arc'){
        self.pen.clearRect(0,0,800,800);//增加代码
        self.pen.drawImage(self.img, 0, 0);//增加代码
        self.pen.beginPath();//增加代码

        if(x < originX){
            newOriginX = x;
        }
        if(y < originY){
            newOriginY = y;
        }
        var r = Math.sqrt(Math.abs(x-originX) * Math.abs(x-originX) + Math.abs(y-originY) * Math.abs(y-originY))
        self.pen.arc(Math.abs(x-originX)+newOriginX, Math.abs(y-originY)+newOriginY , r, 0, 2*Math.PI);
        self.pen.fillStyle = self.color.value;
        self.pen.fill();
        self.pen.closePath();//增加代码
    }
```
Step3:加载js文件
---
```html
    <script src="draw.js"></script>
    <script>
        window.onload = function (){
            var draw = new Draw();
            draw.init();
        };
    </script>
```
最后简单的画板功能就是实现了，如有不足之处请指出
