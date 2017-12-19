> 关于Android桌面小部件的官方教程当然就是[Android开发者文档](https://developer.android.com/guide/topics/appwidgets/index.html)，这里以一个火影迷感兴趣的图腾设计一款桌面时钟，抛砖引玉。

## 效果图
![效果图|from Google Nexus 6P with Screen Size 1440x2560](http://upload-images.jianshu.io/upload_images/2048485-8a44857d2756c09b.gif?imageMogr2/auto-orient/strip)

## 准备素材
### 小部件预览图
![素材-预览图|720x720](http://upload-images.jianshu.io/upload_images/2048485-814b6a4d841f220d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### widget_bg.png
![素材-钟面|720x720](http://upload-images.jianshu.io/upload_images/2048485-b6cff1a301b4b6e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### widget_hour_00.png
![素材-时针|720x720](http://upload-images.jianshu.io/upload_images/2048485-bb71be78e237d2f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### widget_min_00.png
![素材-分针|720x720](http://upload-images.jianshu.io/upload_images/2048485-dcf9d0d5ca54138b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### widget_sec_00.png
![素材-秒针|720x720](http://upload-images.jianshu.io/upload_images/2048485-03673b8b880399bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

四张切图使用AI绘制导出，规格均为720X720，放置于Android工程res/drawable-xxxhdpi目录下，这样时钟大小较合适，原因参见表一及[Android Screen Matching](http://www.jianshu.com/p/c8d4ae30edd0)。

表一

|name|icon size|scope|代表屏幕|scale|
|--:|:--:|:--:|:--:|:--|
|ldpi|36x36|0~120dpi|现今鲜有设备|0.75|
|mdpi|48x48|120~160dpi|320x480|1|
|hddpi|72x72|160~240dpi|480x800|1.5|
|xhdpi|96x96|240~320dpi|720x1280|2|
|xxhdpi|144x144|320~480dpi|1080x1920|3|
|xxxhdpi|192x192|480~640dpi|1440x2560|4|

## 编写时钟布局 app_widget_clock.xml
> 注意，App Widget使用的是RemoteViews，仅支持有限的内置控件，自定义控件一律不支持，并且对控件的操作均要通过RemoteViews的有限方法来执行，接下来我们会用到RemoteViews的setImageViewBitmap方法，留意下面的代码。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--Widget支持的控件-->
<!--FrameLayout-->
<!--LinearLayout-->
<!--RelativeLayout-->
<!--GridLayout-->
<!--AnalogClock-->
<!--Button-->
<!--Chronometer-->
<!--ImageButton-->
<!--ImageView-->
<!--ProgressBar-->
<!--TextView-->
<!--ViewFlipper-->
<!--ListView-->
<!--GridView-->
<!--StackView-->
<!--AdapterViewFlipper-->
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp"
    >

  <!--表盘-->
  <ImageView
      android:id="@+id/background"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center"
      android:src="@drawable/widget_bg"
      />

  <!--秒针-->
  <ImageView
      android:id="@+id/time_s"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center"
      android:src="@drawable/widget_sec_00"
      />

  <!--分针-->
  <ImageView
      android:id="@+id/time_m"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center"
      android:src="@drawable/widget_min_00"
      />

  <!--时针-->
  <ImageView
      android:id="@+id/time_h"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center"
      android:src="@drawable/widget_hour_00"
      />
</FrameLayout>
```

## 编写AppWidgetProvider

### 编写ClockAppWidgetProvider.java
```java
public class ClockAppWidgetProvider extends AppWidgetProvider {

  public ClockAppWidgetProvider() {
    super();
  }

  @Override
  public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
    super.onUpdate(context, appWidgetManager, appWidgetIds);
    // 调用的间隔由res/xml/app_widget_info_clock.xml下的updatePeriodMillis决定
    // 下面的for循环是update app widgets的标准写法
    // N是桌面上该小部件的数目
    final int N = appWidgetIds.length;
    for (int i = 0; i < N; i++) {
      // 对每一个小部件进行更新
      int appWidgetId = appWidgetIds[i];
      RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.app_widget_clock);
      // TODO 对remoteViews进行操作，比如添加点击事件跳转系统时钟
      appWidgetManager.updateAppWidget(appWidgetId, remoteViews);
    }
    // TODO 启动ClockService
  }

  @Override public void onDeleted(Context context, int[] appWidgetIds) {
    super.onDeleted(context, appWidgetIds);
    // TODO 任意一个小部件被移除时调用
  }

  @Override public void onEnabled(Context context) {
  }

  @Override public void onDisabled(Context context) {
    // 所有桌面小部件被移除时调用
    // TODO 注销ClockService
  }
}
```

### 编写res/xml/app_widget_info_clock.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialLayout="@layout/app_widget_clock"
    android:minHeight="250dp"
    android:minWidth="250dp"
    android:previewImage="@drawable/widget_clock_preview"
    android:resizeMode="horizontal|vertical"
    android:updatePeriodMillis="86400000"
    >
</appwidget-provider>
```
- initialLayout初始化布局
- minHeight & minWidth的设置参见[小部件设计](https://developer.android.com/guide/practices/ui_guidelines/widget_design.html#anatomy_determining_size)
```math
// cellCountInRowOrColumn - 小部件的行数或列数
valueInDP = 70 × cellCountInRowOrColumn − 30
```
- previewImage是小部件的预览图，长按桌面查看小部件列表时显示的那个icon就是它
- resizeMode可伸缩的方向
- updatePeriodMillis小部件的刷新间隔，单位是秒，默认是一天

### Manifest的声明
```xml
<receiver android:name=".ui.widget.clock.ClockAppWidgetProvider">
  <intent-filter>
      <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>
  </intent-filter>
  <meta-data
      android:name="android.appwidget.provider"
      android:resource="@xml/app_widget_info_clock"/>
</receiver>
```

## 编写ClockService
> ClockService用于接收系统时间，并根据时间的时分秒值转动我们的时针、分针和秒针。

```math
// 时分秒针角度推导
// 假设rawS、rawM、rawH分别为秒、分、时（12小时制）的数值
// angleS、angleM、angleH分别为秒、分、时针的转动角度
angleS = 360 / 60 × rawS
       = 360 / 60 × realS
其中，令 realS = rawS

angleM = 360 / 60 × rawM + 360 / 3600 × rawS
       = 360 / 60 × (rawM + rawS / 60)
       = 360 / 60 × (rawM + realS / 60)
       = 360 / 60 × realM
其中，令 realM = rawM + realS / 60

angleH = 360 / 12 × rawH + 360 / 12 / 3600 × (60 × rawM + rawS)
       = 360 / 12 × (rawH + rawM / 60 + rawS / 3600)
       = 360 / 12 × (rawH + (rawM + rawS / 60) / 60)
       = 360 / 12 × (rawH + realM / 60)
       = 360 / 12 × realH
其中，令 realH = rawH + realM / 60
```

### 编写时钟任务
```java
private final class MyTimerTask extends TimerTask {

  @Override public void run() {

    // 获取Widgets管理器
    AppWidgetManager widgetManager = AppWidgetManager.getInstance(getApplicationContext());
    // widgetManager所操作的Widget对应的远程视图即当前Widget的layout文件
    RemoteViews remoteView = new RemoteViews(getPackageName(), R.layout.app_widget_clock);

    // 见公式推导
    Calendar calendar = Calendar.getInstance();
    int rawS = calendar.get(Calendar.SECOND);
    int rawM = calendar.get(Calendar.MINUTE);
    int rawH = calendar.get(Calendar.HOUR);

    float realS = rawS;
    float realM = rawM + realS / 60.0f;
    float realH = rawH + realM / 60.0f;

    // 计算时分秒针的角度
    float rotateS = 360f / 60f * realS;
    float rotateM = 360f / 60f * realM;
    float rotateH = 360f / 12f * realH;

    // 根据角度转动时分秒针
    if (null == mHandS || mHandS.isRecycled()) {
      mHandS = BitmapFactory.decodeResource(getApplicationContext().getResources(), R.drawable.widget_sec_00);
    }
    if (null == mHandM || mHandM.isRecycled()) {
      mHandM = BitmapFactory.decodeResource(getApplicationContext().getResources(), R.drawable.widget_min_00);
    }
    if (null == mHandH || mHandH.isRecycled()) {
      mHandH = BitmapFactory.decodeResource(getApplicationContext().getResources(), R.drawable.widget_hour_00);
    }

    // RemoteViews的内置方法操作控件
    // 对内置控件的操作，RemoteViews仅提供有限的几个方法，这里我们用到其中一个：setImageViewBitmap
    remoteView.setImageViewBitmap(R.id.time_s, rotateBitmap(mHandS, rotateS));
    remoteView.setImageViewBitmap(R.id.time_m, rotateBitmap(mHandM, rotateM));
    remoteView.setImageViewBitmap(R.id.time_h, rotateBitmap(mHandH, rotateH));

    // 当点击Widgets时触发的事件
    ComponentName componentName = new ComponentName(getApplicationContext(), ClockAppWidgetProvider.class);
    widgetManager.updateAppWidget(componentName, remoteView);
  }
}
```

### Bitmap旋转
```java
/**
 * 旋转 
 *
 * @param source
 * @param degree from 0f to 360f
 * @return
 */
private Bitmap rotateBitmap(Bitmap source, float degree) {
  if (null == source) {
    return null;
  }
  int size = source.getWidth();
  Matrix matrix = new Matrix();
  matrix.reset();
  matrix.setRotate(degree, size / 2, size / 2);
  return Bitmap.createBitmap(source, 0, 0, size, size, matrix, true);
}
```

### ClockService启动MyTimerTask
```java
public class ClockService extends Service {

  private Timer mTimer;
  
  @Override public void onCreate() {
    super.onCreate();
    mTimer = new Timer();
    // 1000ms执行一次
    mTimer.schedule(new MyTimerTask(), 0, 1000);
  }

  // TODO 其它生命周期方法
}
```
记得在Manifest.xml里声明ClockService
```xml
<service android:name=".ui.widget.clock.ClockService"/>
```

## 源码参考

[Github源码](https://github.com/hayukleung/AppWidget)