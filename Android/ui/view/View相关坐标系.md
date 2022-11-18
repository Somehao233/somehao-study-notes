# 坐标系

## Surface坐标系

```java
public void getLocationInSurface(int[] location)
```

不知道具体是做什么的，但似乎与多屏（折叠屏）设备有关。

## Screen坐标系

```java
public void getLocationOnScreen(int[] outLocation)
```

获取View的左上角相对于屏幕左上角的偏移量。

## Window坐标系

```java
public void getLocationInWindow(int[] outLocation)
```

获取View左上角相对于所处Window左上角的偏移量。

## x, y

```java
public float getX() {
    return mLeft + getTranslationX();
}

public void setX(float x) {
    setTranslationX(x - mLeft);
}

public float getY() {
    return mTop + getTranslationY();
}

public void setY(float y) {
    setTranslationY(y - mTop);
}
```

考虑了translation的局部坐标。

## scrollX, scrollY

View内容的偏移量

## translationX, translationY

View的视觉偏移量，与布局无关。由RenderNode在native层用矩阵实现。

## left,  top

View的左上角相对于父View布局空间左上角的偏移量（不考虑translation）。

View在父View中的实际绘制位置的坐标：

left - parent.scrollX + translationx, top - parent.scrollY + translationY

