# MASK抠图的解决方案

## 背景

H5与小程序活动中时经常需要获取抠好的人像。

在不压缩的前提下，如果用户上传的图片很大，那么服务端返回的图片就会很大，一般为半透明的png图片，比实际用户上传的图片还要大。

这种图片从上传到显示耗时会很长，很影响用户体验。

## 解决方案

服务端只返回用户上传人像对应的MASK图（此图为一张黑白的jpg，大小比返回的半透明抠图要小很多），然后将MASK图与原图对比，使用canvas进行抠图。

## 代码

```js
/**
 * 根据mask抠出图片
 * @param {ImageObject} sourceImg    原始图片
 * @param {ImageObject} maskImg   接口返回的mask图片
 * @return {dataURL} 返回base64的数据流 
 */

export default async function (sourceImg, maskImg) {
    try {

        if (
            typeof sourceImg !== 'object' || sourceImg.tagName !== 'IMG'
            ||
            typeof maskImg !== 'object' || maskImg.tagName !== 'IMG'
        ) {
            throw new Error('参数必须为image对象')
        }

        let imgData = getCtx(sourceImg).getImageData(0, 0, sourceImg.width, sourceImg.height)
        let maskData = getCtx(maskImg).getImageData(0, 0, maskImg.width, maskImg.height)

        if (imgData.data.length !== maskData.data.length) {
            throw new Error('原图与mask图大小不一致')
        }

        function getCtx(imgObj) {
            let canvas = document.createElement('canvas')
            let context = canvas.getContext('2d')
            canvas.width = imgObj.width
            canvas.height = imgObj.height
            context.drawImage(imgObj, 0, 0)
            return context
        }

        // 对比原图与mask，修改imgData中的部分颜色通道为纯透明
        function filter(_imgData, _maskData) {
            for (let i = 0, len = _maskData.length; i < len; i += 4) {
                let r = _maskData[i],
                    g = _maskData[i + 1],
                    b = _maskData[i + 2];
                if (r == 0 && g == 0 && b == 0) {
                    _imgData[i + 3] = 0; // 把白色改成透明的  
                }
            }
        }

        filter(imgData.data, maskData.data)

        let _ctx = getCtx(sourceImg),
            _canvas = _ctx.canvas;

        _ctx.putImageData(imgData, 0, 0)

        return _canvas.toDataURL('image/png', 0.9)

    } catch (error) {
        throw error
    }
}
```