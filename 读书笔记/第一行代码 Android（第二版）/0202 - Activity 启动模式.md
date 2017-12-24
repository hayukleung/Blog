## standard
```xml
<activity 
  android:name="com.hayukleung.DemoActivity"
  android:launchMode="standard"/>
```

## singleTop
```xml
<activity 
  android:name="com.hayukleung.DemoActivity"
  android:launchMode="singleTop"/>
```

## singleTask
```xml
<activity 
  android:name="com.hayukleung.DemoActivity"
  android:launchMode="singleTask"/>
```

## singleInstance
```xml
<activity 
  android:name="com.hayukleung.DemoActivity"
  android:launchMode="singleInstance"/>
```

## Activity 返回栈管理
```java
public class ActivityCollector {
  public static List<Activity> activities = new ArrayList<>();
  
  public static void addActivity(Activity activity) {
    activities.add(activity);
  }
  
  public static void removeActivity(Activity activity) {
    activities.remove(activity);
  }
  
  public static void finishAll() {
    for (Activity activity : activities) {
      if (!activity.isFinishing()) {
        activity.finish();
      }
    }
  }
}
```