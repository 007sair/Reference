# 适配`iphoneX`

## H5端

#### step1

在html内添加如下头：

```html
<meta name="viewport" content="width=device-width, viewport-fit=cover">
```

#### step2

添加对应样式即可。

```scss
@supports (bottom: constant(safe-area-inset-bottom)) or (bottom: env(safe-area-inset-bottom)) {
    body {
        padding-bottom: constant(safe-area-inset-bottom); /* 兼容 iOS < 11.2 */
        padding-bottom: env(safe-area-inset-bottom); /* 兼容 iOS >= 11.2 */
    }
}
```

## 小程序端

#### step1

通过`wx.getSystemInfo`获取到设备机型，做出判断是否是iphonX

#### step2

给特殊元素添加适配iphonX的样式

```html
<!-- mpvue写法 -->
<div :class="{'iphoneX-fixed': isIPX}"></div>
```

```scss
.iphoneX-fixed {
    display: block;
    height: 34rpx; // 34rpx为底部高度
}
```

