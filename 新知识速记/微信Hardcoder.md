# 微信Hardcoder

## Hardcoder是什么

是微信的一个性能优化框架

## 原理

Hardcoder 在 App 和系统之间，构建了一个可靠的通信框架，跳出之前 App 只能调用系统标准 API，而无法直接调用系统底层硬件资源的问题，让 Android App 与 系统之间，可以实现实时的通信。

![Hardcoder](https://mmbiz.qpic.cn/mmbiz_png/liaczD18OicSyGWFnc5mtFzgXSkdlpSPp7aLRfHyWtFt0wBiaHqvN0ZgOjaAetvPBRFZQP1QXHB4F8JsesSciawAGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

利用 Hardcoder，App 可以在必要的时候，向系统申请更多的硬件资源，例如 CPU 频率、大小核、GPU 频率等资源来提升 App 的性能。

参考：
[微信开源了 Hardcoder，旨在解决手机「卡成狗」，但开发者先别高兴](https://mp.weixin.qq.com/s/9uPoKU7fHwvAj9lyGDbsFA)
[github地址](https://github.com/Tencent/Hardcoder)
