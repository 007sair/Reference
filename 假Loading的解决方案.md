# 假Loading的解决方案

## 背景

在制作活动的过程中，由于资源过多过大等原因，需要加入预加载功能。但是由于web的特性，我们并不能预先知道真实资源的大小与传输速度及当前进度，所以无法在预加载时显示实时的进度。

## 解决方案

既然无法真实获取到进度，那么通过一定手段，使用假效果模拟真实进度，以假乱真的方式实现！思路：

1. 不设定加载时间上限，因为总时间本就未知。
2. 每过一段时间自动加载一段进度
3. 真实资源加载完毕（这个状态可以获取到）时立即将进度增长到100%

## 代码

```js
/**
 * 假进度条，用于模拟真实ajax等异步操作
 * Usage:
 * 
    ```js
    // 初始化
    const fp = new FakeProgress({
        onProgress(n) {
            // 业务相关代码
            // me.progress = Math.ceil(n)
        }
    })

    // 发起请求
    fp.start()

    // 异步请求中..

    // 请求成功，关闭loading
    fp.done().then(() => {
        // do somthing
    })

    // 请求失败
    fp.fail()
    ```
 */

class FakeProgress {

    constructor(config) {
        this.config = Object.assign({}, {
            total: 100, // 总进度
            onProgress(progress) {} // 进度监听函数，参数为当前进度
        }, config)

        this.progress = 0
        this.time = 500 // 每次进度变化的时间间隔
        this.step = 0.2 // 每次进度增加幅度
        this.timer = null
        this.onProgress = this.config.onProgress
    }

    // 开始加载
    start() {
        // 动态变化进度每次增长幅度和时间间隔
        this.time = Math.floor(this._rdm(500, 2000))
        this.step = this._rdm(0.08, 0.25)

        this.progress += (this.config.total - this.progress) * this.step
        this.timer = setTimeout(this.start.bind(this), this.time)
        this.onProgress(this.progress)
    }

   /**
    * 加载完成函数
    * @param {boolean/number} async 如果为数字，则表示异步，值为等待时间, false时表示不异步
    */
    done(async = 700) {
        if (typeof async === 'number' && async) {
            return new Promise((resolve) => {
                this._end()
                setTimeout(resolve, async)
            })
        } else {
            this._end()
        }
    }

    // 加载失败时触发
    fail() {
        this.progress = 0
        clearTimeout(this.timer)
    }

    // 私有方法
    _end() {
        clearTimeout(this.timer)
        this.progress = this.config.total
        this.onProgress(this.progress)
    }

    // 私有方法
    // 取区间随机数，包含小数
    _rdm(min, max) {
        return Math.random() * (max - min) + min;
    }

}

export default FakeProgress

```