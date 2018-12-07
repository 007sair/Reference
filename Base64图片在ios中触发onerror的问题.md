# `Base64`图片在ios中触发`onerror`的问题

## 背景

在`H5`中使用`canvas`时，经常使用到`Base64`图，原因在于`canvas`有相关能力将`canvas`转换为`Base64`图，而这种图片无法直接渲染到画布中，需要我们像`load`一个`http`的图像一样进行`load`后才能渲染到`canvas`中。

而某些低版本`(8.3-11.0)`的`ios`系统中，触发`img.onerror`事件。原因为：浏览器对`Base64`的长度有一些限制问题，参考[这里](https://stackoverflow.com/questions/21728604/ie10-base64-encoded-image-load-error)。

## 解决方案

使用`URL.createObjectURL`代替`Base64`

## 伪代码

```javascript
/**
 * 异步加载图片
 * @param {string}   url         图片地址
 * @param {boolean}  is_revoke   是否删除blob对象，默认删除
 * @param {boolean}  is_cors     是否对此元素的CORS请求设置凭据标志
 * @return {imageObject}        img对象
 */
 function loadImage(url, is_revoke = true, is_cors = true) {
    return new Promise((resolve, reject) => {
        if (typeof url !== 'string') {
            throw new TypeError('url 类型错误')
        }
        let img = new Image()
        if (is_cors) {
            img.crossOrigin = 'Anonymous'
        }

        let objectURL = null
        if (url.match(/^data:(.*);base64,/) && window.URL && URL.createObjectURL) {
            objectURL = URL.createObjectURL(this.dataURL2blob(url))
            url = objectURL
        }

        img.onload = () => {
            if (is_revoke) {
                objectURL && URL.revokeObjectURL(objectURL)
            }
            resolve(img)
        }
        img.onerror = () => {
            reject(new Error('That image was not found.:' + url.length))
        }
        img.src = url
    })
}
```