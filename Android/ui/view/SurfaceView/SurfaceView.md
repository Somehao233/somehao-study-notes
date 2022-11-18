# SurfaceView

## 官方介绍文档翻译

提供了一个嵌入到View树里的专门用来绘制的Surface。开发者可以自由控制这个Surface的格式和尺寸。SurfaceView会负责把Surface正确地放置在屏幕里地某个位置。

## 个人理解

### SurfaceHolder

每个SurfaceView会在构造时就创建一个SurfaceHolder，这个SurfaceHolder是个匿名内部类实例，并且是final的。外部可以调用getHolder()直接获取到这个SurfaceHolder。

```java
private final SurfaceHolder mSurfaceHolder = new SurfaceHolder() {
    /* 省略内部代码 */
}

public SurfaceHolder getHolder() {
    return mSurfaceHolder;
}
```

外部应该通过SurfaceHolder向SurfaceView注册回调：

```java
/* SurfaceView */

final ArrayList<SurfaceHolder.Callback> mCallbacks = new ArrayList<>();

/* SurfaceHolder */

public void addCallback(Callback callback) {
    synchronized (mCallbacks) {
        if (!mCallbacks.contains(callback)) {
            mCallbacks.add(callback);
        }
    }
}

public void removeCallback(Callback callback) {
    synchronized (mCallbacks) {
        mCallbacks.remove(callback);
    }
}
```

外部可以通过SurfaceHolder控制SurfaceView的宽高：

```java
/* SurfaceView */

int mRequestedWidth = -1;
int mRequestedHeight = -1;

/* SurfaceHolder */
public void setFixedSize(int width, int height) {
    /* 省略内部代码 */
}

public void setSizeFromLayout() {
    /* 省略内部代码 */
}
```

setFixedSize和setSizeFromLayout都会修改mRequestXX的值，并调用requestLayout请求重新布局。

setFixedSize会设置固定的宽高，setSizeFromLayout会把mRequestXX设置成-1，表示把SurfaceView就当作一个普通View测量。

SurfaceView在onMeasure中并不会无脑采用mRequestXX的值，仍然会尊重MeasureSpec提供的约束。

外部可以通过SurfaceHolder控制SurafceView的像素格式：

```java
public void setFormat(int format) {
    /* 省略内部代码 */
}
```

这个方法会修改mRequestedFormat的值，并调用updateSurface。
### SurfaceHolder.Callback