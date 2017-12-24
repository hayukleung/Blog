Android 6.0 引入了运行时权限

## 权限分类

### 普通权限 Normal Permissions
> 只需要在AndroidManifest.xml中声明即可

- `ACCESS_LOCATION_EXTRA_COMMANDS`
- `ACCESS_NETWORK_STATE`
- `ACCESS_NOTIFICATION_POLICY`
- `ACCESS_WIFI_STATE`
- `BLUETOOTH`
- `BLUETOOTH_ADMIN`
- `BROADCAST_STICKY`
- `CHANGE_NETWORK_STATE`
- `CHANGE_WIFI_MULTICAST_STATE`
- `CHANGE_WIFI_STATE`
- `DISABLE_KEYGUARD`
- `EXPAND_STATUS_BAR`
- `GET_PACKAGE_SIZE`
- `INSTALL_SHORTCUT`
- `INTERNET`
- `KILL_BACKGROUND_PROCESSES`
- `MODIFY_AUDIO_SETTINGS`
- `NFC`
- `READ_SYNC_SETTINGS`
- `READ_SYNC_STATS`
- `RECEIVE_BOOT_COMPLETED`
- `REORDER_TASKS`
- `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`
- `REQUEST_INSTALL_PACKAGES`
- `SET_ALARM`
- `SET_TIME_ZONE`
- `SET_WALLPAPER`
- `SET_WALLPAPER_HINTS`
- `TRANSMIT_IR`
- `UNINSTALL_SHORTCUT`
- `USE_FINGERPRINT`
- `VIBRATE`
- `WAKE_LOCK`
- `WRITE_SYNC_SETTINGS`

### 危险权限 Dangerous Permissions
> 危险权限基本都涉及到用户的隐私，诸如拍照、读取短信、写存储、录音等，Android系统将这些危险权限分为9组，获取分组中某个权限的同时也就获取了同组中的其他权限，危险权限不仅需要在AndroidManifest.xml中注册，还需要动态的申请权限。

| Permission Group | Permissions
| ---------------- | -------------
| CALENDAR         | `READ_CALENDAR`
|                  | `WRITE_CALENDAR`
| CAMERA           | `CAMERA`
| CONTACTS         | `READ_CONTACTS`
|                  | `WRITE_CONTACTS` 
|                  | `GET_ACCOUNTS`
| LOCATION         | `ACCESS_FINE_LOCATION`
|                  | `ACCESS_COARSE_LOCATIO`
| MICROPHONE       | `RECORD_AUDIO`
| PHONE            | `READ_PHONE_STATE`
|                  | `CALL_PHONE`
|                  | `READ_CALL_LOG`
|                  | `WRITE_CALL_LOG` 
|                  | `ADD_VOICEMAIL`
|                  | `USE_SIP`
|                  | `PROCESS_OUTGOING_CALLS`
| SENSORS          | `BODY_SENSORS`
| SMS              | `SEND_SMS`
|                  | `RECEIVE_SMS`
|                  | `READ_SMS`
|                  | `RECEIVE_WAP_PUSH`
|                  | `RECEIVE_MMS`
| STORAGE          | `READ_EXTERNAL_STORAGE`
|                  | `WRITE_EXTERNAL_STORAGE`

何时需要动态申请权限？
1. 危险权限
2. Android 版本 >= 6.0
3. targetSdkVersion >= 23

三个条件缺一不可

### 特殊权限 Special Permissions
> 看权限名就知道特殊权限比危险权限更危险，特殊权限需要在manifest中申请并且通过发送Intent让用户在设置界面进行勾选

- `SYSTEM_ALERT_WINDOW`：设置悬浮窗
- `WRITE_SETTINGS`：修改系统设置

```java
// 申请SYSTEM_ALERT_WINDOW权限
private static final int REQUEST_CODE = 1;
private void requestAlertWindowPermission() {
  Intent intent = newIntent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
  intent.setData(Uri.parse("package:" + getPackageName()));
  startActivityForResult(intent, REQUEST_CODE);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  super.onActivityResult(requestCode, resultCode, data);
  if (requestCode == REQUEST_CODE) {
    if (Settings.canDrawOverlays(this)) {
      Log.i(LOGTAG,"onActivityResult granted");
    }
  }
}
```

```
// 申请WRITE_SETTINGS权限
private static final int REQUEST_CODE_WRITE_SETTINGS = 2;
private void requestWriteSettings() {
  Intent intent = newIntent(Settings.ACTION_MANAGE_WRITE_SETTINGS);
  intent.setData(Uri.parse("package:" + getPackageName()));
  startActivityForResult(intent, REQUEST_CODE_WRITE_SETTINGS);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  super.onActivityResult(requestCode, resultCode, data);
  if (requestCode == REQUEST_CODE_WRITE_SETTINGS) {
    if (Settings.System.canWrite(this)) {
      Log.i(LOGTAG,"onActivityResult write settings granted");
    }
  }
}
```

## 相关 API

为方便开发者实现权限管理，Google提供了4个API：

| API                                    | 作用
|----------------------------------------|---
| checkSelfPermission()                  | 判断权限是否具有某项权限
| requestPermissions()                   | 申请权限
| onRequestPermissionsResult()           | 申请权限回调方法
| shouldShowRequestPermissionRationale() | 是否要提示用户申请该权限的缘由

### `shouldShowRequestPermissionRationale`

|序号|是否授予了权限|ssrpr()返回|是否勾选不再询问
|-------|----|-------|---
| 1     | 否 | false | -
| 2     | 否 | true  | 否
| 3     | 否 | true  | 否
| …     | …	 | …     | …
| i     | 否 | true  | 是
| i + 1 | -  | false | -


