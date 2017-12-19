## 老规矩，效果图先上

![我是效果图](http://upload-images.jianshu.io/upload_images/2048485-a117cf21db73302c.gif?imageMogr2/auto-orient/strip)
> 点击输入框，软键盘弹起
> 点击Switch按钮时，软键盘与面板进行切换
> 触碰空白处，软键盘与面板均隐藏

这个效果的关键点是获取软键盘的高度（我们用变量KeyboardHeight代表软键盘的高度），将面板的高度设置为KeyboardHeight，那么切换时，输入框在屏幕的位置就岿然不动了，看似简单，实际上要获取KeyboardHeight要花些许功夫。

先上布局。
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_chat_room"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.hayukleung.kps.ChatRoomActivity"
    >
  <ScrollView
      android:id="@+id/kps_content"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:layout_above="@+id/kps_bottom"
      >
  </ScrollView>
  <LinearLayout
      android:id="@+id/kps_bottom"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout_alignParentBottom="true"
      android:orientation="vertical"
      >
    <RelativeLayout
        android:id="@+id/kps_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
      <EditText
          android:id="@+id/kps_text"
          android:layout_width="match_parent"
          android:layout_height="40dp"
          android:layout_centerVertical="true"
          android:layout_toLeftOf="@+id/kps_switch_panel"
          />
      <Button
          android:id="@+id/kps_switch_panel"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_alignParentEnd="true"
          android:layout_centerVertical="true"
          android:text="switch\npanel and keyboard"
          android:textSize="10sp"
          />
    </RelativeLayout>
    <View
        android:id="@+id/kps_panel"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:background="@color/colorPrimary"
        />
  </LinearLayout>
</RelativeLayout>
```

![Android Screen Size](http://upload-images.jianshu.io/upload_images/2048485-a6b2c883005172a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可以得出KeyboardHeight与屏幕各部分尺寸值得算术关系，我们逐一将它们计算出来。

## 软键盘的弹起
首先我们要了解Android软键盘弹起时Activity的布局变化情况，设置SOFT_INPUT_ADJUST_RESIZE，Activity的布局会根据软键盘的高度压缩自己，设置SOFT_INPUT_ADJUST_PAN，Activity的布局只会保证输入框焦点出的位置不被软键盘遮挡，将布局往上推**最小**的距离。我们计算软键盘的做法就是利用**SOFT_INPUT_ADJUST_RESIZE**的特点，软键盘弹起时Activity设置为SOFT_INPUT_ADJUST_RESIZE。这样就可以得到Activity的布局高度的变化值，这个值就与软键盘的高度有关了。

```java
/**
 * 显示键盘
 */
private void showKeyboard() {
  mInputMethodManager.showSoftInput(mKpsText, SHOW_IMPLICIT);
  setSoftInputMode(SOFT_INPUT_STATE_UNCHANGED | SOFT_INPUT_ADJUST_RESIZE);
}
/**
 * 隐藏键盘
 */
private void hideKeyboard() {
  mInputMethodManager.hideSoftInputFromWindow(mKpsText.getWindowToken(), HIDE_NOT_ALWAYS);
  setSoftInputMode(SOFT_INPUT_STATE_UNCHANGED | SOFT_INPUT_ADJUST_PAN);
}
/**
 * 动态更改软键盘模式
 *
 * Desired operating mode for any soft input area. May be any combination of:
 *
 * <ul>
 * <li> One of the visibility states
 * {@link #SOFT_INPUT_STATE_UNSPECIFIED},
 * {@link #SOFT_INPUT_STATE_UNCHANGED},
 * {@link #SOFT_INPUT_STATE_HIDDEN},
 * {@link #SOFT_INPUT_STATE_VISIBLE},
 * {@link #SOFT_INPUT_STATE_ALWAYS_HIDDEN},
 * {@link #SOFT_INPUT_STATE_ALWAYS_VISIBLE}.
 * <li> One of the adjustment options
 * {@link #SOFT_INPUT_ADJUST_NOTHING},
 * {@link #SOFT_INPUT_ADJUST_UNSPECIFIED},
 * {@link #SOFT_INPUT_ADJUST_RESIZE},
 * {@link #SOFT_INPUT_ADJUST_PAN}.
 *
 * @param mode
 */
private void setSoftInputMode(int mode) {
  getWindow().setSoftInputMode(mode);
}
```
要监听Activity高度的变化，可以对Activity的根布局设置OnGlobalLayoutListener监听。一旦布局的大小发生变化就会执行里面的onGlobalLayout方法，我们会在该方法完成对软键盘高度的计算。一开始软键盘未弹起时，该方法会一直空转，一旦弹起，我们就拿到了数据完成计算，完成了计算及时移除该监听，避免不必要的性能损耗。

```java
// mActivityChatRoom是根布局Layout，软键盘弹起时会引起它的布局大小变化，对它进行布局变化监听
mActivityChatRoom.getViewTreeObserver()
    .addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
      private int keyboardHeight = 0;
      private boolean hasNavigationBar = true;
      @Override public void onGlobalLayout() {
        // TODO 计算StatusBarHeight
        // TODO 计算ActionBarHeight
        // TODO 计算NavigationBarHeight
        // TODO 计算ScreenHeight
        // TODO 计算ActivityHeight
        // TODO 计算KeyboardHeight
        if (0 < keyboardHeight) {
          // 成功计算得到KeyboardHeight，及时移除监听
          mActivityChatRoom.getViewTreeObserver()
            .removeOnGlobalLayoutListener(this);
        }
      }
    });
```

## StatusBarHeight
```java
Rect rect = new Rect();
getWindow().getDecorView().getWindowVisibleDisplayFrame(rect);
int statusBarHeight = rect.top;
```
## ActionBarHeight
```java
TypedValue tv = new TypedValue();
getTheme().resolveAttribute(R.attr.actionBarSize, tv, true);
int actionBarHeight = TypedValue.complexToDimensionPixelSize(tv.data, getResources().getDisplayMetrics());
```

## NavigationBarHeight
```java
Resources resources = getResources();
int navigationBarHeight = resources.getDimensionPixelSize(resources.getIdentifier("navigation_bar_height","dimen", "android"));
```
计算NavigationBarHeight时需要注意，我们的手机如果不存在NavigationBar，这个值算出来并不是0，所以不能简单的通过该值判断手机是否存在NavigationBar，需要进行一步额外的判断。具体是，键盘未弹起时计算得到的KeyboardHeight应该是0，如果计算出来的值是-NavigationBarHeight，说明该手机没有NavigationBar，这时候将NavigationBar不存在的布尔状态记录下来，等到软键盘弹起计算高度时就根据该布尔量忽略掉NavigationBarHeight。


## ScreenHeight & ActivityHeight
```java
// mActivityChatRoom是根布局Layout
int screenHeight = mActivityChatRoom.getRootView().getHeight();
int activityHeight = mActivityChatRoom.getHeight();
```

## KeyboardHeight
```java
keyboardHeight = screenHeight - activityHeight - statusBarHeight - actionBarHeight - navigationBarHeight;
if (0 == keyboardHeight + navigationBarHeight) {
  // 不存在navigation bar
  hasNavigationBar = false;
}
keyboardHeight += hasNavigationBar ? 0 : navigationBarHeight;
ViewGroup.LayoutParams params = mKpsPanel.getLayoutParams();
params.height = keyboardHeight;mKpsPanel.setLayoutParams(params);
```

[GitHub源码](https://github.com/hayukleung/KeyboardPanelSwitch)