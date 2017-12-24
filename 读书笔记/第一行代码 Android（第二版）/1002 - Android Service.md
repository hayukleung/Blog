## Service

- `onCreate`：服务创建时执行
- `onStartCommand`：服务启动时执行，使用 startService 方式启动服务时，Service 的回调
- `onBind`：使用 bindService 方式绑定服务时，Service 的回调
- `onDestroy`：服务销毁时执行，stopService\stopSelf\unbindService
- `stopSelf`：服务自己执行停止

## IntentService
`onHandleIntent`

