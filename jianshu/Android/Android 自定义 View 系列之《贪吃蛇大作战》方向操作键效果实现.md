> 前段时间很火的一款贪吃蛇游戏，可玩性很高，几点规则改造就将传统的贪吃蛇改活了，当时我拿过13000多分，还嘚瑟了很久。今天来个教程10分钟实现它。。。额，不是，实现它的方向操作按钮效果，看下图左下角的那两个同心圆。

![贪吃蛇大作战](http://upload-images.jianshu.io/upload_images/2048485-6fc0b39290a90ac8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 用户手指触碰屏幕任意位置，内圆就往用户手指那个方向移动至外圆边界内切，实现后效果图如下所示。

![效果图](http://upload-images.jianshu.io/upload_images/2048485-dd807d22eeface06.gif?imageMogr2/auto-orient/strip)

先看两张图，分别是Android坐标系与Android View尺寸函数的含义，其中，Android坐标系往右x轴递增，往下y轴递增，不多说。

![Android坐标系](http://upload-images.jianshu.io/upload_images/2048485-a8687291cb3ee3b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Android-View-Size](http://upload-images.jianshu.io/upload_images/2048485-cffd4d3f7c6d23fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面开始编码

1）创建HandleView类，继承自View

```java
/**
 * 贪吃蛇大作战方向控制按钮效果
 */
public class HandleView extends View {

  public HandleView(Context context) {
    this(context, null);
  }

  public HandleView(Context context, AttributeSet attrs) {
    this(context, attrs, 0);
  }

  public HandleView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
  }

  @Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    // TODO
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
  }

  @Override protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    super.onLayout(changed, left, top, right, bottom);
  }

  @Override protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // TODO
  }
}
```
上述代码可以作为几乎所有自定义View的初始代码模板。

方向操作按钮等宽等高，我们不想它在xml布局时被设置成宽高不等的长方形，所以需要在onMeasure函数里进行处理。

2）重载onMeasure

```java
@Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  setMeasuredDimension(getDefaultSize2(getSuggestedMinimumWidth(), widthMeasureSpec),
  getDefaultSize2(getSuggestedMinimumHeight(), heightMeasureSpec));
  int childWidthSize = getMeasuredWidth();
  widthMeasureSpec = MeasureSpec.makeMeasureSpec(childWidthSize, MeasureSpec.EXACTLY);
  heightMeasureSpec = MeasureSpec.makeMeasureSpec(childWidthSize, MeasureSpec.EXACTLY);
  super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```

```java
/**
 * Compare to: {@link android.view.View#getDefaultSize(int, int)}
 * If mode is AT_MOST, return the child size instead of the parent size
 * (unless it is too big).
 */
private static int getDefaultSize2(int size, int measureSpec) {
  int result = size;
  int specMode = MeasureSpec.getMode(measureSpec);
  int specSize = MeasureSpec.getSize(measureSpec);
  
  switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
      result = size;
      break;
    case MeasureSpec.AT_MOST:
      result = Math.min(size, specSize);
      break;
    case MeasureSpec.EXACTLY:
      result = specSize;
      break;
  }
  return result;
}
```

3）画外圆

```java
public class HandleView extends View {
  private Paint mPaintForCircle;
  // ...
  @Override protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // 背景透明
    canvas.drawColor(Color.TRANSPARENT);
    // 外圆半径
    int radiusOuter = getWidth() / 2;
    // 内圆半径
    int radiusInner = getWidth() / 5;

    // 圆心坐标(cx,cy)
    float cx = getWidth() / 2;
    float cy = getHeight() / 2;

    if (null == mPaintForCircle) {
      mPaintForCircle = new Paint();
    }
    mPaintForCircle.setAntiAlias(true);
    mPaintForCircle.setStyle(Paint.Style.FILL);

    // 画外圆
    mPaintForCircle.setColor(Color.argb(0x7f, 0x11, 0x11, 0x11));
    canvas.drawCircle(cx, cy, radiusOuter, mPaintForCircle);
    // TODO 画内圆
  }
}
```

4）画内圆

内圆是运动的，它的位置与用户的手指触摸坐标有关，按照效果，用户可以触摸的范围是包裹HandleView的ViewGroup（FrameLayout之类的），这里先写个接口用于获取手指触摸坐标。

```java
public class HandleView extends View {

  private HandleReaction mHandleReaction;

  public void setHandleReaction(HandleReaction handleReaction) {
    mHandleReaction = handleReaction;
  }

  public interface HandleReaction {
    /**
     * 获取用户触摸坐标
     * @return
     */
    float[] getTouchPosition();
  }
  
  // ...
}
```

![示意图](http://upload-images.jianshu.io/upload_images/2048485-87467c8f54a71c25.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
内圆半径固定，位置由圆心的坐标决定，所以关键是得出内圆的圆心坐标随用户手指的触摸坐标的变化而变化的函数关系，让我们建立方程式：（用工具画太花时间，将就手画，见谅！P.S.好像回到中学有木有）

![建立方程组](http://upload-images.jianshu.io/upload_images/2048485-22cf2f5843e87888.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![结果](http://upload-images.jianshu.io/upload_images/2048485-9fb5d7d0da6cfd37.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由公式可得出，分母（开平方根那个数的值）是cx2和cy2都需要的公用的值，命名为ratio，计算代码如下。

```java
float[] touchPosition = mHandleReaction.getTouchPosition();
double ratio = (radiusOuter - radiusInner) / 
  Math.sqrt(
    Math.pow(touchPosition[0] - cx, 2) + 
    Math.pow(touchPosition[1] - cy, 2));
float cx2 = (float) (ratio * (touchPosition[0] - cx) + cx);
float cy2 = (float) (ratio * (touchPosition[1] - cy) + cy);

mPaintForCircle.setColor(Color.argb(0xff, 0x11, 0x11, 0x11));
canvas.drawCircle(cx2, cy2, radiusInner, mPaintForCircle);
```

5）获取触摸坐标
建立MainActivity，布局文件就不给出了，文末附带源码地址。

```java
public class MainActivity extends AppCompatActivity
    implements HandleView.HandleReaction, View.OnTouchListener {

  private float[] mTouchPosition = null;
  private HandleView mHandleView;

  @Override protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    FrameLayout frameLayout = (FrameLayout) findViewById(R.id.frameLayout);
    frameLayout.setOnTouchListener(this);
    mHandleView = (HandleView) findViewById(R.id.handleView);
    mHandleView.setHandleReaction(this);
  }

  @Override public boolean onTouch(View view, MotionEvent motionEvent) {
    switch (motionEvent.getAction()) {
      case MotionEvent.ACTION_DOWN:
      case MotionEvent.ACTION_MOVE: {
        mTouchPosition = new float[2];
        mTouchPosition[0] = motionEvent.getX();
        mTouchPosition[1] = motionEvent.getY();
        mHandleView.invalidate();
        return true;
      }
      case MotionEvent.ACTION_UP: {
        mTouchPosition = null;
        mHandleView.invalidate();
        return true;
      }
    }
    return false;
  }

  @Override public float[] getTouchPosition() {
    return mTouchPosition;
  }
}
```

6）坐标修正
表面上，上面内圆圆心计算代码是正确的，但实际上，由于我们的HandleView通过接口从它的父布局那里拿到了触摸坐标与HandleView内部坐标的参考坐标系不是同一个，他们相差一个HandleView相对于它父布局的getLeft与getTop的偏移，参照图Android-View-Size，所以需要对计算代码进行修正，如下：

```java
// 经过修正后的内圆圆心坐标代码
double ratio = (radiusOuter - radiusInner) / 
  Math.sqrt(
    Math.pow(touchPosition[0] - cx - getLeft(), 2) + 
    Math.pow(touchPosition[1] - cy - getTop(), 2));
float cx2 = (float) (ratio * (touchPosition[0] - cx - getLeft()) + cx);
float cy2 = (float) (ratio * (touchPosition[1] - cy - getTop()) + cy);
```

最后附上源码地址
[GitHub源码](https://github.com/hayukleung/HandleView)