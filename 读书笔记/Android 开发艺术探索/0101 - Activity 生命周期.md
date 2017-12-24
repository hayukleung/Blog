1. `onStart`与`onStop`是从 Activity 是否可见的角度回调
`onResume`与`onPause`是从 Activity 是否在前台的角度回调

2. 从 Activity A 中启动 Activity B，Activity A 的`onPause`先执行，然后 Activity B 才会被启动，所以在`onPause`和`onStop`回调中
    - 不能执行耗时大的操作，尤其是`onPause`
    - 尽量在`onStop`中执行

3. Activity 异常重建
![](http://ooun8fyfu.bkt.clouddn.com/17-5-15/90301024-file_1494832108856_68d8.png)

4. `onSaveInstanceState`在`onStop`之前回调
5. `onRestoreInstanceState`在`onStart`之后回调
6. 数据的恢复可以选择`onCreate`或`onRestoreInstanceState`，区别是在`onCreate`中需要对 bundle 进行判空，在`onRestoreInstanceState`中的 bundle 一定不为空
7. 关于 Activity Manifest 配置中 `android:configChanges` 的配置项：
    - orientation：当屏幕发生旋转时，Activity 不重建
    - 其它见下表
![android:configChanges](http://ooun8fyfu.bkt.clouddn.com/17-5-15/61292450-file_1494833641807_bba4.png)
    - 当配置了`android:configChanges`后，对应的配置发生变化时，Activity 就不会回调`onSaveInstanceState`，而是回调`onConfigurationChanged`




