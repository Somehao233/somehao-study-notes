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

可以通过SurfaceHolder获取Canvas：

```java
Canvas lockCanvas();
Canvas lockCanvas(Rect dirty);
Canvas lockHardwareCanvas();
```

这两个方法都可以获取一个canvas。

两个lockCanvas获取的都是基于Skia库实现的Canvas，是CPU基于执行软件指令绘制的。

lockHardwareCanvas获取的是有硬件加速的Canvas，是用GPU绘制的。

用Canvas完成绘制后，需要将其返还：

```java
void unlockCanvasAndPost(Canvas canvas);
```

最佳写法：

```java
final Canvas canvas = surfaceHolder.lockXXCanvas();
try {
    /* 具体绘制过程 */
} finally {
    surfaceHolder.unlockAndPost(canvas);
}
```


### SurfaceHolder.Callback

这是SurfaceView提供的一个接口：

```java
public interface Callback {
    void surfaceCreated(SurfaceHolder holder);
    void surfaceChanged(SurfaceHolder holder, int format, int width, int height);
    void surfaceDestroyed(SurfaceHolder holder);
}
```

后面还出了个Callback2加了个surfaceRedrawNeeded：

```java
public interface Callback2 extends Callback {
    void surfaceRedrawNeeded(SurfaceHolder holder)
}
```

这些接口的名字已经告诉了你它们各自的用途是什么。它们都是在updateSurface方法里伺机调用的。

surfaceCreated表示新的Surface已经创建完毕，此时可以分配一些资源，如开启绘制线程。

surfaceChanged表示Surface配置发生变化，此时要重新调整可能收宽高、颜色格式影响的参数。

surfaceRedrawNeeded表示SurfaceView里的内容需要重新绘制，这个通常是伴随surfaceChanged调用。所以重新绘制的事情放在这里面做就行了，不要在surfaceChanged里面做绘制工作。

surfaceDestroyed表示之前的Surface已经销毁了，这时候要做一些资源释放等收尾工作。在新的Surface准备好之前，都不要再使用已经销毁的Surface。

显然surfaceChanged和surfaceRedrawNeeded是在surfaceCreated和surfaceDestroyed之间调用的。