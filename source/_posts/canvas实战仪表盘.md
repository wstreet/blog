---
title: canvas实战仪表盘
date: 2020-07-07 08:54:23
tags: canvas
categories: canvas
---


[完整代码](https://github.com/wstreet/canvas/blob/master/dashboard/index.html)<br />
[可视化学习](https://www.yuque.com/streetex/fbqzli)
![GIF 2020-5-30 13-06-01.gif](https://cdn.nlark.com/yuque/0/2020/gif/211977/1590815221568-be0220ca-71f2-4bfa-9880-c83180259e27.gif#align=left&display=inline&height=142&margin=%5Bobject%20Object%5D&name=GIF%202020-5-30%2013-06-01.gif&originHeight=381&originWidth=476&size=124070&status=done&style=none&width=177)
<br />有动画效果
```javascript
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <canvas id="canvas" height="400" width="400"></canvas>
</body>
<script>
    const canvas = document.querySelector('#canvas')
    const ctx = canvas.getContext('2d')

    let initValue = 0
    const value = 91 // 钱多的程度 0-100
    const valueText = getValueText(value)

    // 修正角度
    const updateAngle = 135 / 180 * Math.PI


    function getValueText(value) {
        if (value <= 0) {
            return '太穷了'
        } else if (value <= 50) {
            return '钱很少'
        } else if (value <= 70) {
            return '钱不多'
        } else if (value <= 90) {
            return '一点点'
        } else if (value <= 100) {
            return '首富'
        } else {
            return '成仙了'
        }
    }

    function getAngle(value) {
        if (value < 0) {
            value = 0
        }

        if (value > 100) {
            value = 100
        }
        const angle = value / 100 * 270 / 180 * Math.PI
        return angle
    }

    const requestAnimationFrame = window.requestAnimationFrame
        || window.webkitRequestAnimationFrame
        || window.mozRequestAnimationFrame
        || function (fn) { setTimeout(fn, 16.7) }

    // 初始化位置
    ctx.translate(200, 200)

    function draw() {
        if (initValue < value) {
            initValue += 1
        }

        // 计算旋转角度
        const angle = getAngle(initValue)

        ctx.clearRect(-canvas.width / 2, -canvas.width / 2, canvas.width, canvas.height)

        // 保存初始样式
        ctx.save()

        ctx.rotate(updateAngle)

        // 第二个环
        ctx.strokeStyle = '#f5f9fc'
        ctx.lineWidth = 40
        ctx.lineCap = 'butt'
        ctx.beginPath()
        ctx.arc(0, 0, 50, 0, 270 / 180 * Math.PI)
        ctx.stroke()

        // 内环
        ctx.fillStyle = '#feffff'
        ctx.shadowOffsetX = 0;
        ctx.shadowOffsetY = 0;
        ctx.shadowBlur = 10;
        ctx.shadowColor = "#eceef2";
        ctx.beginPath()
        ctx.arc(0, 0, 40, 0, Math.PI * 2)
        ctx.fill()
        ctx.rotate(-updateAngle)

        // valueText
        ctx.font = '12px serif';
        ctx.fillStyle = '#1f2348'
        ctx.fillText(valueText, -12, 16)
        ctx.save()

        ctx.rotate(updateAngle)

        // 外环 加动画
        const lineargradient = ctx.createLinearGradient(100, 300, 300, 100);
        lineargradient.addColorStop(0, '#8fe9d7');
        lineargradient.addColorStop(1, '#21d9b4');
        ctx.strokeStyle = lineargradient
        ctx.lineWidth = 20 // lineWidth一分为二，里外各占一半
        ctx.lineCap = 'round'

        ctx.beginPath()

        ctx.arc(0, 0, 80, 0, angle)
        ctx.stroke()

        // 三角 加动画
        ctx.beginPath()
        ctx.rotate(angle)
        ctx.translate(40, 0)
        ctx.fillStyle = '#abb2c1'
        ctx.moveTo(0, -6)
        ctx.lineTo(8, 0)
        ctx.lineTo(0, 6)
        ctx.closePath()
        ctx.fill()

        ctx.restore()
        ctx.fillStyle = '#1f2348'
        ctx.font = '30px serif';
        ctx.fillText(initValue, -12, 0)

        requestAnimationFrame(draw)

    }
    draw()

</script>

</html>
```
