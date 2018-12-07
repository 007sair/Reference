# 低版本ios的图片展示问题

## 背景

在某些ios版本(8.3-10.3)的系统中，给图片设置垂直居中时图片会消失。代码如下：

```html
<!-- mpvue写法 -->
<div class="poster">
    <img :src="poster" mode="aspectFit" />
</div>
```

```scss
.poster {
    display: flex;
    justify-content: center;
    align-items: center;
    flex: 1;
    > img {
        max-width: 100%;
        max-height: 100%;
        height: 100%;
    }
}
```

经过测试，发现其外层容器宽高仍然存在，但图片就是不显示。

## 解决方案

既然flex不起作用，那么就将显示方式换成定位。

```scss
.poster {
    position: relative;
    flex: 1;
    > img {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        max-width: 100%;
        max-height: 100%;
        height: 100%;
    }
}
```