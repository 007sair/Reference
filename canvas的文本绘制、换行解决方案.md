# canvas的文本绘制、换行解决方案

最初为了尝试在画布中操作文本，写了个[DEMO](https://github.com/007sair/canvas_text/)。

## 背景

在制作贴纸活动时，需要在图片上创建贴纸文字，文字由用户输入、修改，再被添加到对应位置。且文字可变色、拖拽、缩放功能。

## 解决方案

创建贴纸文字对象，包含如下属性：

```js
const canvas_data = {
	text: '测试测试Test',
	font_size: 40,
	sx: 0,
	sy: 0,
	angle: 0,
	fake_scale: 100
}
```

思路：

1. 定义文字大小、颜色、字体，就像css一样
2. 计算偏移、缩放、旋转相关数值，这些数值来自于事件中，如拖拽事件改变了translate的值
3. 使用fillText填充当前文本到canvas中

核心绘制函数如下：

```js
/**
 * 绘制canvas
 * @param {canvas} canvas 需要绘制的canvas对象
 * @param {number} canvas_scale 当前预览canvas与生成海报的canvas之间的比例大小
 */
function draw(canvas, canvas_scale = 1) {
    let ctx = canvas.getContext('2d')

    // 这个scale才是真实的，data里面的scale是为了range控件放大了100倍
    let scale = this.canvas_data.fake_scale / 100

    let font_size = this.canvas_data.font_size * canvas_scale

    ctx.fillStyle = "red"
    ctx.fillRect (0, 0, canvas.width, canvas.height)

    ctx.fillStyle = '#fff'
    ctx.textBaseline = 'top'
    ctx.font = `${font_size}px 宋体` //设置字体

    let text_info = ctx.measureText(this.canvas_data.text)

    let sx = +this.canvas_data.sx * canvas_scale,
        sy = +this.canvas_data.sy * canvas_scale

    let translate_x = text_info.width / 2 + sx,
        translate_y = (font_size * 1.2) / 2 + sy

    ctx.translate(translate_x, translate_y);
    ctx.rotate(this.canvas_data.angle * Math.PI / 180);
    ctx.scale(scale, scale);
    ctx.translate(-translate_x, -translate_y);
    ctx.fillText(this.canvas_data.text, sx, sy);

    return canvas
}
```

## 换行

本换行只针对手动换行，自动换行有其自己的实现方式且不在本业务范围内。

手动换行一般用于用户输入文本的过程中按了回车键的操作。

手动跟单行的区别在于多行有高度计算问题，思路：

1. 获取字号、行高
2. 通过换行符\n将单行文本切割成多行，这里的\n为人为添加（手动的原因）
3. 找出最长的那行文本后计算出宽度
4. 计算偏移量等信息
5. 绘制出文本

核心代码如下：

```js
/**
 * 绘制贴纸文字
 * @param {CanvasContext} ctx 画布上下文 2d
 * @param {Object} cell 单个元件数据
 */
_drawText(ctx, cell) {
    let { translate, stickerText } = cell
    // 将字号放大
    let font_size = stickerText.font_size * this.scale
    // 通过字号获取行高
    let lineHeight = font_size * stickerText.line_height
    // 按照换行符，切割原始文本
    let result = stickerText.text.split('\n')
    // 先设置文本样式，再获取到的宽度才是正确的
    ctx.textBaseline = 'top'
    //设置字体
    ctx.font = `${font_size}px "Helvetica Neue", Helvetica, Arial, "PingFang SC", "Hiragino Sans GB", "Heiti SC", "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif` 
    // 找到所有文本中最长的那条数据
    let longestLine = result.slice(0).sort((m, n) => n.length - m.length)[0]
    // measureText：必须在设置样式后调用
    let longestLineWidth = ctx.measureText(longestLine).width
    // 计算文本起始位置
    let sx = (translate.sx + stickerText.padding) * this.scale,
        sy = (translate.sy + stickerText.padding) * this.scale,
        translate_x = longestLineWidth / 2 + sx,
        translate_y = (lineHeight * result.length) / 2 + sy;

    ctx.save()
    ctx.translate(translate_x, translate_y)
    ctx.rotate(translate.rotation * Math.PI / 180)
    ctx.scale(translate.scale, translate.scale)
    ctx.translate(-translate_x, -translate_y)
    // 逐行绘制文本
    result.forEach(function (line, index) {
        ctx.fillStyle = stickerText.color
        ctx.fillText(line, sx, sy + lineHeight * index)
    })
    ctx.restore()
}
```
