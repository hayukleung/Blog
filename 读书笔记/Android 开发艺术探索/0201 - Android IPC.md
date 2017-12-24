[toc]

## Android APP 多进程的设置方法

1. 设置Manifest 里的`android:process`
    1. APP 私有进程`android:process=":xxx"`
    2. 全局进程`android:process="package-name.xxx"`，其它 APP 可以利用 shareUID 与之跑在同一进程

2. 利用 JNI 在 C 代码里 fork 新进程

## 序列化
1. Serializable
2. Parcelable

## IPC 方式

![](http://ooun8fyfu.bkt.clouddn.com/17-5-15/29502050-file_1494852982524_3f96.png)

### Bundle
### 共享文件
### Messenger
![](http://ooun8fyfu.bkt.clouddn.com/17-5-15/30947787-file_1494852541539_1066b.png)
### AIDL
### ContentProvider
### Socket