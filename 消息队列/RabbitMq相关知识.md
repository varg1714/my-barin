### 线程中断

当 Rabbit 的监听器中跑出了以下异常时，线程将会认为发生致命错误而中断，具体实现参考 ConditionalRejectingErrorHandler 中的 isFatal 方法。
![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20211213130613.png)