---
title: ScrollView 与 ViewDragHelper 的混合使用学习
date: 2017-01-06 20:20:02
tags: Android
---

按照惯例，先上效果

![](http://7xv11f.com1.z0.glb.clouddn.com/suppriseBottomView.gif)

### 涉及到的知识点

* `ViewGroup` 的事件分发拦截机制

* `ViewDragHelper` 的基本使用


#### `ViewGroup` 的事件分发拦截机制

`ViewGroup` 有三个方法

1. `dispatchTouchEvent`
2. `onInterceptTouchEvent`
3. `onTouchEvent`

##### dispatchTouchEvent

用于`touch`事件的分发；通俗点说，就是决定当前这个`touch`事件应该交给谁来处理（是当前`View`还是父`View`）

> 当触摸事件发生时 `Activity` 的 `dispatchTouchEvent(MotionEvent ev)` 方法会从根元素依次往下传递直到最内层子元素或在中间某一元素中事件被拦截或者消费.
>
> 如果 `return true`，事件会分发给当前 `View` 并由 `dispatchTouchEvent` 方法进行消费，同时事件会停止向下传递；这样该`View`的`onTouchEvent`事件也不会得到响应.
> 如果` return false`，会将事件返回给父 `View` 的 `onTouchEvent` 进行消费。
> 如果 `return super.dispatchTouchEvent(ev)`，事件会分发给当前 `View` 的 `onInterceptTouchEvent` 方法去进行处理。

##### onInterceptTouchEvent

用于`touch`事件的拦截；通俗点说，就是决定刚刚 `dispatchTouchEvent` 抛给我的`touch`事件应该交给谁来处理（是当前`View`还是子`View`）*注意跟`dispatchTouchEvent`的区别*

> 如果 `return true`，则将事件进行拦截，并将拦截到的事件交由该 `View` 的 `onTouchEvent` 进行处理；
> 如果 `return false`，则将事件向子`View`传递，再由子` View `的 `dispatchTouchEvent `来对这个事件处理；
> 如果 `return super.onInterceptTouchEvent(ev)`，事件会被拦截，并将事件交由该 `View` 的 `onTouchEvent` 进行处理。

##### onTouchEvent

用于touch事件的处理；通俗点说，就是决定刚刚 onInterceptTouchEvent 抛给我的touch事件进行处理（~~这锅实在没得甩了~~ 还可以甩给父类，科科）

> 如果`return false`，那么这个事件会从该 `View` 向父`View`传递，父 `View` 的 `onTouchEvent` 来接收，而且如果父`View`也是`return false`，那事件也会向上传递由`onTouchEvent`接收处理.
> 如果`retrun true`， 则会接收并消费该事件。
> 如果`retrun super.onTouchEvent(ev) ` 和返回 false 时相同。

#### ViewDragHelper 的基本使用

1. 创建实例
2. `Touch` 事件分发拦截的相关处理
3. `ViewDragHelper callback` 的实现

`ViewDragHelper` 的出现就是为了减轻 开发者实现拖拽行为功能 的负担，所以很简单的几个方法就可以做出~~很好的效果~~比以前用`onTouchEvent`自己写得要死要活的效果还要好。

<!-- more -->

##### 创建实例

```java
mDragViewHelper = ViewDragHelper.create(this, 1.0f, mDragHelperCallback)
```
##### `Touch` 事件分发拦截的相关处理

既然出动了`ViewDragHelper`，所以，很多的触摸事件我们就要放手给TA去处理了。

```java
/**
* 事件分发
* @param ev
* @return
* 可以根据具体需求决定如何分发事件
× 这里不做处理
*/
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    Log.w("jimbray", "dispatchTouchEvent");
    return super.dispatchTouchEvent(ev);
}

/**
* 事件拦截 
* @param ev
* @return
* 直接将所有的事件都交由 ViewDragHelper 去处理
* 可以根据具体需求决定是否交付事件
*/
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    Log.w("jimbray", "onInterceptTouchEvent");
    return mDragViewHelper.shouldInterceptTouchEvent(ev);
}

/**
* 事件消费
* @param event
* @return
* 直接将所有的事件都交由 ViewDragHelper 去处理
* 可以根据具体需求决定是否交付事件
* 交付事件后直接 return true，结束事件
*/
@Override
public boolean onTouchEvent(MotionEvent event) {
    Log.w("jimbray", "onTouchEvent");
    mDragViewHelper.processTouchEvent(event);
  return true;
}
```
##### `ViewDragHelper callback` 的实现

```java
private ViewDragHelper.Callback mDragHelperCallback = new ViewDragHelper.Callback() {
        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            return child == mDragView;
        }

        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
          	int topBound = -1 * bottomView.getMeasuredHeight();
            int bottomBound = getHeight() - mDragView.getMeasuredHeight();
            return Math.min(Math.max(top, topBound), bottomBound);
        }

        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            int leftBound = mDragView.getLeft();
            int rightBound = mDragView.getRight() - mDragView.getWidth();
            return Math.min(Math.max(leftBound, left), rightBound);
        }

        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            if(releasedChild == mDragView) {
                mDragViewHelper.settleCapturedViewAt(mDragViewOri.x, mDragViewOri.y);
                postInvalidate();
            }
        }

        @Override
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) 		   {
            if(changedView == mDragView) {
                bottomView.layout(bottomView.getLeft(), 
                top + mDragView.getHeight(), 
                bottomView.getRight(), top+mDragView.getHeight() +              bottomView.getMeasuredHeight());
            }
            postInvalidate();
        }
  
  		@Override
        public int getViewVerticalDragRange(View child) {
            return getMeasuredHeight() - child.getMeasuredHeight();
        }

        @Override
        public int getViewHorizontalDragRange(View child) {
            return getMeasuredWidth() - child.getMeasuredWidth();
        }
    };
```
`ViewDragHelper.Callback` 有很多方法，这里只说明一下用到的好了。

* tryCaptureView

  如果返回`ture`则表示可以捕获该`view`，你可以根据传入的第一个`view`参数(`child`) 决定哪些可以捕获

* clampViewPositionVertical, clampViewPositionHorizontal

  可以在该方法中对`child`移动的边界进行控制，`left` , `top` 分别为即将移动到的位置

  比如我想要在横向方向进行限制，只让捕获的`View`从坐标系里的 100 移动到 400，可以这样写

  ```java
  int leftBound = 100; // 左边限定为 100

  int rightBound = 400 - mDragView.getWidth(); // 右边限定为 400的位置，由于要使用 left 进行布局，											    // 所以需要减去mDragView的宽度

  return Math.min(Math.max(leftBound, left), rightBound);
  ```

* onViewReleased

  当拖拽`View`的手指释放的时候回调，可以做回弹操作，或者其他需求

* onViewPositionChanged

  当拖拽的`View`位置变化的时候回调

* getViewVerticalDragRange, getViewHorizontalDragRange

  这俩是个坑，当`ViewGroup`内的子控件会消耗事件时（就是有点击或者其他事件时），你会发现`ViewDragHelper`不起作用了。原因是：

  如果子`View`不消耗事件，那么整个手势流程都是直接进入`onTouchEvent`，在`onTouchEvent`的`DOWN`的时候就确定了`captureView`；

  如果消耗事件，那么就会先走`onInterceptTouchEvent`方法，判断是否可以捕获，而在判断的过程中会去判断另外两个回调的方法：`getViewHorizontalDragRange`和`getViewVerticalDragRange`，只有这两个方法返回大于0的值才能正常的捕获。

  上面的写法 我只是为了保证 返回值 大于 0，其实只要`return`大于0的值就可以了，比如直接写 1

* ViewDragHelper 的 settleCapturedViewAt 方法

  回到指定的`left`，` top`的位置。紧随其后的代码是`invalidate()`;因为其内部使用的是`Scroller.startScroll`，所以需要`invalidate()`以及结合`computeScroll`方法一起。

  敲黑板

  敲黑板

  敲黑板

  ```java
  @Override
  public void computeScroll() {
      if(mDragViewHelper.continueSettling(true)) {
        postInvalidate();
      }
  }
  ```

ViewDragHelper 的基本使用流程就到这里了。



### 思路整理

#### 需求分析

> *两个`View`*线性*竖向*布局，上面的`View`内容可以进行*上下滚动*，滚动到*底部的时候*，可以*继续上拉*，第二个`View`*跟随*第一个`View`上拉出现，拉动到第二个`View`的高度后*停止上拉*，*松手*后第二个`View`*回弹*消失，*此时下拉*，仍然可以*向上滚动*第一个`View`的内容

#### 根据思路选择合适的ViewGroup

> 根据上面 *斜体* 文字的~~划重点~~ 提示
>
> 竖向 ： LinearLayout
>
> 上下滚动：ScrollView

#### 根据选择的ViewGroup决定实现方式

> 只有两个View，要么ScrollView里面套LinearLayout，要么LinearLayout里面套ScrollView
>
> 我选的是 前者

那，动手吧。

### 动手实现

先来 `LinearLayout`

```java
public class JimPullBottomView extends LinearLayout {

    private ViewDragHelper mDragViewHelper;

    private View mDragView;
    private View bottomView;

    private Point mDragViewOri = new Point();

    public JimPullBottomView(Context context) {
        super(context, null);
        init(context);
    }

    public JimPullBottomView(Context context, AttributeSet attrs) {
        super(context, attrs, -1);
        init(context);
    }

    public JimPullBottomView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }

    private void init(Context context) {
        mDragViewHelper = ViewDragHelper.create(this, 1.0f, mDragHelperCallback);
    }

    private ViewDragHelper.Callback mDragHelperCallback = new ViewDragHelper.Callback() {
        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            return child == mDragView;
        }

        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {

            int topBound = -1 * bottomView.getMeasuredHeight();

            int bottomBound = getHeight() - mDragView.getMeasuredHeight();

            return Math.min(Math.max(top, topBound), bottomBound);
        }

        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {

          	// 左边限定为 100
            int leftBound = 100; 
			// 右边限定为 400的位置，由于要使用 left 进行布局,所以需要减去mDragView的宽度
            int rightBound = 400 - mDragView.getWidth(); /

            return Math.min(Math.max(leftBound, left), rightBound);
        }

        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            if(releasedChild == mDragView) {
                mDragViewHelper.settleCapturedViewAt(mDragViewOri.x, mDragViewOri.y);
                postInvalidate();
            }
        }

        @Override
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
            if(changedView == mDragView) {
                bottomView.layout(bottomView.getLeft(), top + mDragView.getHeight(), bottomView.getRight(), top+mDragView.getHeight() + bottomView.getMeasuredHeight());
            }
            postInvalidate();
        }

        @Override
        public int getViewVerticalDragRange(View child) {
            return getMeasuredHeight()-child.getMeasuredHeight();
        }

        @Override
        public int getViewHorizontalDragRange(View child) {
            return getMeasuredWidth()-child.getMeasuredWidth();
        }
    };

        @Override
        public void computeScroll() {
            if(mDragViewHelper.continueSettling(true)) {
              postInvalidate();
            }
        }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);

        mDragView.layout(getPaddingLeft(), getPaddingTop(), getWidth() - getPaddingRight(), mDragView.getMeasuredHeight());
        bottomView.layout(getPaddingLeft(), mDragView.getBottom(), getWidth() - getPaddingRight(), mDragView.getBottom() + bottomView.getMeasuredHeight());

        mDragViewOri.x = mDragView.getLeft();
        mDragViewOri.y = mDragView.getTop();
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mDragView = getChildAt(0);
        bottomView = getChildAt(1);
    }

    /**
    * 事件分发
    * @param ev
    * @return
    * 可以根据具体需求决定如何分发事件
    × 这里不做处理
    */
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.w("jimbray", "dispatchTouchEvent");
        return super.dispatchTouchEvent(ev);
    }

    /**
    * 事件拦截 
    * @param ev
    * @return
    * 直接将所有的事件都交由 ViewDragHelper 去处理
    * 可以根据具体需求决定是否交付事件
    */
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.w("jimbray", "onInterceptTouchEvent");
        return mDragViewHelper.shouldInterceptTouchEvent(ev);
    }

    /**
    * 事件消费
    * @param event
    * @return
    * 直接将所有的事件都交由 ViewDragHelper 去处理
    * 可以根据具体需求决定是否交付事件
    * 交付事件后直接 return true，结束事件
    */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
    	Log.w("jimbray", "onTouchEvent");
        mDragViewHelper.processTouchEvent(event);
      return true;
    }

    private int dp2Px(float dp) {
        final float scale = getContext().getResources().getDisplayMetrics().density;
        return (int) (dp * scale + 0.5f);
    }
}

```

好了，先来看看几个点。

#### 两个View

```java
private View mDragView;
private View bottomView;
```

#### 竖向布局

````java
mDragView.layout(getPaddingLeft(), 
                 getPaddingTop(), 
                 getWidth() - getPaddingRight(), 
                 mDragView.getMeasuredHeight());
bottomView.layout(getPaddingLeft(), 
                  mDragView.getBottom(), 
                  getWidth() - getPaddingRight(), 
                  mDragView.getBottom() + bottomView.getMeasuredHeight());
````

在`onLayout`直接布局 使用 layout 方法进行位置布局

该方法是`View`的放置方法，在`View`类实现。调用该方法需要传入放置`View`的矩形空间左上角left、top值和右下角right、bottom值。*这四个值是相对于父控件而言的*。例如传入的是（10, 10, 100, 100），则该View在距离父控件的左上角位置(10, 10)处显示，显示的大小是长宽是(100-10=90) （参数r,b是相对左上角的）

#### 第二个View 的跟随出现

```java
if(changedView == mDragView) {
	bottomView.layout(bottomView.getLeft(), 
              top + mDragView.getHeight(), 
              bottomView.getRight(), 
              top+mDragView.getHeight() + bottomView.getMeasuredHeight());
}
postInvalidate();
```

第一个`View` 是 `mDragView`, 第二个`View` 是 `bottomView`

既然要跟随出现，那肯定要知道~~第一个View~~`mDragView`的位置变化，所以这段代码放在了 `onViewPositionChanged`里面实现，简单理解一下：

四个参数分别代表：

````java
changedView : 当前被捕获的 View;
left : View 最终处于的位置的左边位置的坐标;
top : View 最终处于的位置的顶部位置的坐标;
dx : 此次调用距离上次调用时的 x 坐标的 差值;
dy : 此次调用距离上次调用时的 y 坐标的 差值;
````

如果移动位置的`View`是~~第一个`View`~~`mDragView`的话，~~第二个`View~~`bottomView`的位置布局为

left : 左边还是一样的

top: 紧紧跟随`mDragView`的底部，因为要粘在`mDragView`的后面嘛

right: 右边也是保持自己的位置

bottom: 只要距离top 自身的高度就可以了

**噢，别忘了 `invalidate()`(与 `postInvalidate()`的区别不在讨论范围了)**

#### 停止上拉

居然有停止上拉的时候，~~难道不是随便拉的吗~~，也就是说，拖动时有范围的，而且效果也看得出来，其实左右是没有办法拖动的。轮到 `clampViewPositionVertical` 和 `clampViewPositionHorizontal` 出场了。

```java
@Override
public int clampViewPositionVertical(View child, int top, int dy) {
  	int topBound = -1 * bottomView.getMeasuredHeight();
    int bottomBound = getHeight() - mDragView.getMeasuredHeight();
    return Math.min(Math.max(top, topBound), bottomBound);
}

@Override
public int clampViewPositionHorizontal(View child, int left, int dx) {
  	int leftBound = mDragView.getLeft();
  	int rightBound = mDragView.getRight() - mDragView.getWidth();
  	return Math.min(Math.max(leftBound, left), rightBound);
}
```

* 先来看下横向限制

左边限制是 `mDragView.getLeft()`就是 `View`的左边

右边限制是 `mDragView.getRight() - mDragView.getWidth()` 就是 `View`的右边

也就是说，这个`View`往左只能到 自己的左边，往右只能到自己的右边，不就是不能移动嘛~~真可怜~~

* 现在看看竖向限制

上边限制是 `-1 * bottomView` 就是屏幕网上 bottomView的高度

下边限制是 `getHeight() - mDragView.getMeasuredHeight()` 就是 屏幕高度最底部的位置，也就是无法移动到底部以上了

#### 松手回弹

```java
if(releasedChild == mDragView) {
	mDragViewHelper.settleCapturedViewAt(mDragViewOri.x, mDragViewOri.y);
    postInvalidate();
}
```

如果松手的`View`是`mDragView`的话，回弹到 `mDragView`最原始的 `left` 和 `top` 位置。

但，那是哪里呢？那我们就在布局之后记录一下。

```java
mDragViewOri.x = mDragView.getLeft();
mDragViewOri.y = mDragView.getTop();
```

写完了，那直接套上用用吧。先布个局

```java
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <xyz.jimbray.custom_view.view.JimPullBottomView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:background="#000">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="center"
            android:orientation="vertical">

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A0"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A1"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A2"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A3"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A4"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A5"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A6"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A7"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A8"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#0af"
                android:gravity="center"
                android:text="A9"
                android:textColor="@android:color/white" />

            <TextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_margin="10dp"
                android:background="#ff00cc"
                android:gravity="center"
                android:text="A10"
                android:textColor="@android:color/white" />

        </LinearLayout>

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="260dp"
            android:background="#EFEF0F">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="260dp"
                android:layout_gravity="center"
                android:adjustViewBounds="true"
                android:scaleType="center"
                android:src="@mipmap/icon" />
        </RelativeLayout>
    </xyz.jimbray.custom_view.view.JimPullBottomView>

</ScrollView>
```

看看效果怎么样

![](http://7xv11f.com1.z0.glb.clouddn.com/surpriseBottomView-unfinished-1.gif)

为什么会这样呢？刚刚那么辛苦算的拖动到位置什么的，都不生效了！摔

仔细想想，滑动得怎么欢快，应该是`ScrollView` 自己处理的所有的 `touch` 事件，完全没有交给 `ViewDragHelper` 去处理嘛（不交给TA处理，我们写的东西当然没效果啦）。那这里就要学习到 ` touch` 事件的拦截处理了。

首先先继承一下 `ScrollView`

```java
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d("jimbray", "scrollview dispatchTouchEvent");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d("jimbray", "scrollview onInterceptTouchEvent");
        int action = ev.getActionMasked();
        switch(action) {
            case MotionEvent.ACTION_DOWN:
                lastY = ev.getY();
                break;

            case MotionEvent.ACTION_MOVE:
                float curY = ev.getY();
                int deltaY = (int) (curY - lastY);
                lastY = curY;

                if(deltaY < 0) { //手指往上划动
                    if(isScrollToBottom()) {
                        return false;
                    }
                }
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        Log.d("jimbray", "scrollview onTouchEvent");
        return super.onTouchEvent(ev);
    }
```

`dispatchTouchEvent` 和 `onTouchEvent` 都是调用 `super` ,其实不重载也没有关系（这个例子里），

关键在于 **事件拦截** **`onInterceptTouchEvent`** 的处理

为什么要处理拦截事件呢？现在遇到的问题就是 `ScrollView`将所有的 `touch`事件都拦截了，子`View`里面根本没有办法施展拳脚，所以我们要做的事情就是 让 `ScrollView` 在适当的时候交出控制权，其他时候还是 交给 `ScrollView `爱怎么滚怎么滚。那现在问题来了：什么是适当的时候呢？

**上面的`View`内容可以进行*上下滚动*，滚动到*底部的时候*，可以*继续上拉*，第二个`View`*跟随*第一个`View`上拉出现**

#### 上下滚动

说明这个时候是交给 `ScrollView` 自己去处理的。因为上下滚动这个活对于 `ScrollView` 来说已经驾轻就熟了

#### 滚动到底部的时候

这个时候继续上拉这个活，就是我们刚刚 在 `LinearLayout` 做的事情了。

现在明白了什么时候 我们的 `LinearLayout` 要从 `ScrollView` 里面抢活，可以开始了。

```java
case MotionEvent.ACTION_DOWN:
    lastY = ev.getY();
    break;

case MotionEvent.ACTION_MOVE:
    float curY = ev.getY();
    int deltaY = (int) (curY - lastY);
    lastY = curY;

    if(deltaY < 0) { //手指往上划动
      	if(isScrollToBottom()) {
        	return false;
      	}
    }
```

这里要结合 上面 事件分发机制的 学习笔记来看。

```java
return false;
```

与

```java
return super.onInterceptTouchEvent(ev);
```

分别代表了什么意义。

主要是对`DOWN`跟`MOVE` 事件的处理

`DOWN`的时候记录按下位置

`MOVE`的时候判断是手指往上划还是往下划

只有往上划的时候才有可能会使用到我们的`LinearLayout`

仅仅有往上划还不行，如果没有划动到 `ScrollView` 的最底部，那`touch`事件应该还是交给 `ScrollView` 来滚

当两个条件同时满足的时候，我们的 `ViewDragHelper` 就要出场了。

现在把 `ScrollView` 换成我们刚写的 `JimPullScrollView`，看效果了。

![](http://7xv11f.com1.z0.glb.clouddn.com/surpriseBottomView-unfinished-1.gif)

这什么鬼？！不是贴错图了吧，怎么跟刚刚一模一样，白写了。

别急，效果是一样的，可以 `touch` 事件的拦截可是生效了的，不信把 `JimPullScrollView` 和 `JimPullBottomView` 的事件拦截方法就打印个日志看看。~~我可是从来不骗人的。咦，我为什么要划掉~~

那是什么原因导致 scrollView 把全部都显示出来了呢？

嗯~/托腮，因为 `ScrollView` 默认显示的就是 子 `View`的高度啊（包括mDragView 和 bottomView）

那我把 子`View`的高度限定一下吧。

```java
@Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        contentView.layout(getPaddingLeft(), 
                getPaddingTop(), 
                getWidth()-getPaddingRight(), 
                contentView.getHeight() - (bottomView.getMeasuredHeight()));
    }
```

`layout`重出江湖，`left`, `top`, `right` 就不说了，还是按照原先的方式来。

主要看下 `bottom`的位置

`contentView.getHeight() - (bottomView.getMeasuredHeight())`

`contentView`的高度 减去 `bottomView` 的高度 = ?

`contentView` 就是 我们的 `LinearLayout`，高度是 `mDragView` 和`bottomView` 的总和

最终结果显而易见 ? = mDragView 的高度

这样的话，`SrcrollView`的显示就只有 `mDragView` 的高度了。

想要看见 bottomView ，就要通过 ViewDragHelper 的上拉处理了。

完成

```java
package xyz.jimbray.custom_view.view;

import android.content.Context;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ScrollView;

/**
 * Created by Jimbray  .
 * on 2017/1/4
 * Email: jimbray16@gmail.com
 * Description: TODO
 */

public class JimPullScrollView extends ScrollView {

    private View contentView;
    private View bottomView;

    private float lastY;

    public JimPullScrollView(Context context) {
        super(context, null);
        init(context);
    }

    public JimPullScrollView(Context context, AttributeSet attrs) {
        super(context, attrs, -1);
        init(context);
    }

    public JimPullScrollView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }

    private void init(Context context) {
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d("jimbray", "scrollview dispatchTouchEvent");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d("jimbray", "scrollview onInterceptTouchEvent");
        int action = ev.getActionMasked();
        switch(action) {
            case MotionEvent.ACTION_DOWN:
                lastY = ev.getY();
                break;

            case MotionEvent.ACTION_MOVE:
                float curY = ev.getY();
                int deltaY = (int) (curY - lastY);
                lastY = curY;

                if(deltaY < 0) {
                    if(isScrollToBottom()) {
                        return false;
                    }
                }

                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        Log.d("jimbray", "scrollview onTouchEvent");
        return super.onTouchEvent(ev);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        contentView.layout(getPaddingLeft(), 
                   getPaddingTop(), 
                   getWidth()-getPaddingRight(), 
                   contentView.getHeight() - (bottomView.getMeasuredHeight()));
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        contentView = getChildAt(0);
        //在这里获取bottomView
        bottomView = ((ViewGroup)contentView).getChildAt(1);
    }

    /*
     * 判断是否划动到了底部 
     */
    private boolean isScrollToBottom() {
        return getScrollY() + getHeight() >= computeVerticalScrollRange();
    }

    private int dp2Px(float dp) {
        final float scale = getContext().getResources().getDisplayMetrics().density;
        return (int) (dp * scale + 0.5f);
    }
}
```

通过这个效果的实现，学习了` touch` 事件的拦截，以及`ViewDragHelper` 的基本使用。

感觉自己棒棒哒