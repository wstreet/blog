---
title: svg基本形状元素
date: 2020-05-07 10:07:03
tags: SVG
categories: SVG
---

# 基本形状元素

基本形状元素允许的内容物：任意数量、任意排序的动画元素、描述性元素<br />
<br />`circle`: svg元素，是一个svg的基本形状，用来创建一个圆，指定圆心和半径<br />允许的内容物：任意数量和任意排序的动画元素、描述性元素<br />属性：cx，cy，r，cx和cy指定圆心，r指定半径<br />例子：
```xml
<?xml version="1.0"?>
<svg viewBox="0 0 120 120" version="1.1"
  xmlns="http://www.w3.org/2000/svg">
  <circle cx="60" cy="60" r="50"/>
</svg>
```
> 

> viewBox的四个参数分别代表：最小X轴数值；最小y轴数值；宽度；高度。



 `ellipse`：svg基本形状，创建一个椭圆，基于一个中心坐标，以及x半径和y半径<br />允许的内容物：任意数量和任意排序的动画元素、描述性元素<br />属性：cx，cy，rx，ry<br />例子：
```xml
<svg xmlns="http://www.w3.org/2000/svg" width="120" 
     height="120" viewPort="0 0 120 120" version="1.1">

    <ellipse cx="60" cy="60" rx="50" ry="25"/>

</svg>
```

<br />`line`元素是一个SVG基本形状，用来创建一条连接两个点的线。<br />属性：x1,x2,y1,y2<br />例子：
```xml
<svg xmlns="http://www.w3.org/2000/svg">
    <line x1="20" y1="100" x2="100" y2="100" stroke-width="2" stroke="black"/>
</svg>
```
> stroke-width：线，文本或元素轮廓厚度，stroke：线，文本或元素轮廓颜色


<br />`polygon` :元素定义了一个由一组首尾相连的直线线段构成的闭合多边形形状。最后一点连接到第一点。<br />属性：point<br />例子：
```xml
<svg xmlns="http://www.w3.org/2000/svg" width="120" height="120" 
     viewPort="0 0 120 120" version="1.1">
    <polygon points="60,20 100,40 100,80 60,100 20,80 20,40"/>

</svg>
```

<br />`polyline` :创建一系列直线连接多个点，一个非闭合（最后一点不与第一点相连）的形状<br />属性：point<br />例子：
```xml
<svg xmlns="http://www.w3.org/2000/svg" width="120" height="120" 
     viewPort="0 0 120 120" version="1.1">

    <polyline fill="none" stroke="black" points="20,100 40,60 70,80 100,20"/>

</svg>
```

<br />`rect` :用来创建一个矩形，基于一个角位置以及它的宽和高。它还可以用来创建圆角矩形。<br />属性：x, y, width, height, rx, ry<br />例子：
```xml
// 简单矩形
<?xml version="1.0"?>
<svg width="120" height="120"
     viewBox="0 0 120 120"
     xmlns="http://www.w3.org/2000/svg">

  <rect x="10" y="10" width="100" height="100"/>
</svg>

// 圆角矩形
<?xml version="1.0"?>
<svg width="120" height="120"
     viewBox="0 0 120 120"
     xmlns="http://www.w3.org/2000/svg">

  <rect x="10" y="10"
        width="100" height="100"
        rx="15" ry=""/>

</svg>
```


