---
title: 我的第一个自定义View
date: 2016-07-13 12:35:05
tags: Android
---
直接上代码，做个学习笔记好了。

```
/**
 * Created by Jimbray  .
 * on 2016/7/12
 * Email: jimbray16@gmail.com
 * Description: 这是我第一个真正意义上的自定义View
 */
public class MyFirstProgressbar extends View {

    //max progress 虽然注释写了，但是还是不解释
    private int mMaxProgress = 100;

    // current progress
    private int mCurrentProgress;

    //progress bar 背景颜色
    private int mBackgroundColor;

    //progress bar 进度条颜色
    private int mProgressColor;

    //progress bar 进度条高度
    private float mProgressBarheight;

    //progress bar 背景高度
    private float mBackgroundHeight;

    //设置默认值
    //默认progress bar 背景颜色
    private final int default_background_color = Color.rgb(204, 204, 204);
    //默认progress bar 进度条颜色
    private final int default_progressbar_color = Color.rgb(66, 145, 241);
    //默认 pregress bar 背景高度
    private float default_background_height;
    //默认 porgress 进度条高度
    private float default_progressbar_height;
    //默认progress bar 背景宽度（最小宽度）
    private final float default_background_width = dp2px(45.f);

    //画笔
    private Paint mBackgroundPaint, mProgressbarPaint;

    //progress 需要draw 的矩形区域
    private RectF mBackgroundRectF = new RectF(0, 0, 0, 0);
    private RectF mProgressbarRectF = new RectF(0, 0, 0, 0);


    public MyFirstProgressbar(Context context) {
        super(context);
        init(context, null, 0);
    }

    public MyFirstProgressbar(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs, 0);
    }

    public MyFirstProgressbar(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);

    }

    private void init(Context context, AttributeSet attrs, int defStyleAttr) {
        default_background_height = dp2px(1.0f);
        default_progressbar_height = dp2px(1.5f);
        //progress bar 背景高度 比 progress bar 进度条高度 矮啊

        //都是用默认值，没有把这些数值抛出去做属性
        mBackgroundHeight = default_background_height;
        mProgressBarheight = default_progressbar_height;

        mBackgroundColor = default_background_color;
        mProgressColor = default_progressbar_color;

        initPainters();
    }

    /**
     * 初始化 画笔 Paint
     */
    private void initPainters() {
        mBackgroundPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mBackgroundPaint.setColor(mBackgroundColor);

        mProgressbarPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mProgressbarPaint.setColor(mProgressColor);
    }

    /**
     * 重载 函数
     * @return view 的推荐最小宽度
     * 修改为 progress bar 背景的默认高度
     */
    @Override
    protected int getSuggestedMinimumWidth() {
        return (int) default_background_width;
    }

    /**
     * 重载 函数
     * @return view 的推荐最小高度
     * 修改为 progress bar 的默认高度 （娶个最高的吧，不然高的那个头被截掉了就不好了）
     */
    @Override
    protected int getSuggestedMinimumHeight() {
        return Math.max((int) mBackgroundHeight, (int) mProgressBarheight);
    }

    /**
     * 这个函数的 解析 Google 一大把，就不解释了
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     * 现在这里是个套路活，跟着老司机走就行了
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(measure(widthMeasureSpec, true), measure(heightMeasureSpec, false));
    }

    /**
     * 测量 view 的 宽高信息
     * @param measureSpec
     * @param isWidth
     * @return
     */
    private int measure(int measureSpec, boolean isWidth) {
        int result;
        int mode = MeasureSpec.getMode(measureSpec);
        int size = MeasureSpec.getSize(measureSpec);
        //不要忘记 padding ，万一有人设置了呢
        int padding = isWidth ? getPaddingLeft() + getPaddingRight() : getPaddingTop() + getPaddingBottom();
        if (mode == MeasureSpec.EXACTLY) { //如果 width 或者 height 都设置了 精确值：比如 45dp啊什么的
            result = size; //就按主人的来吧，诶，森么是主人
        } else {
            //其他情况，既然你都不知道要什么，就按我的套路来吧
            result = isWidth ? getSuggestedMinimumWidth() : getSuggestedMinimumHeight();
            result += padding; //padding 这个小伙子还是很调皮的
            if (mode == MeasureSpec.AT_MOST) {
                if (isWidth) {
                    result = Math.max(result, size);
                } else {
                    result = Math.min(result, size);
                }
            }
        }
        return result;
    }

    /** 要开车了，快上车！
     * @param canvas
     */
    @Override
    protected synchronized void onDraw(Canvas canvas) {
        calRectF();

        canvas.drawRect(mBackgroundRectF, mBackgroundPaint);

        canvas.drawRect(mProgressbarRectF, mProgressbarPaint);
    }

    /**
     * 精华就在这里了。
     * 一开始我也看不懂
     * 现在我也看不懂
     * 可是效果出来了
     * 就这么滴吧
     * 多看几次
     * 说不定就懂了呢
     */
    private void calRectF() {
        mProgressbarRectF.left = getPaddingLeft();
        mProgressbarRectF.top = getHeight() / 2.0f - mProgressBarheight / 2.0f;
        mProgressbarRectF.right = (getWidth() - getPaddingLeft() - getPaddingRight()) / (getMax() * 1.0f) * getProgress() + getPaddingLeft();
        mProgressbarRectF.bottom = getHeight() / 2.0f + mProgressBarheight / 2.0f ;

        mBackgroundRectF.left = mProgressbarRectF.right;
        mBackgroundRectF.top = getHeight() / 2.0f - mBackgroundHeight / 2.0f;
        mBackgroundRectF.right = getWidth() - getPaddingRight();
        mBackgroundRectF.bottom = getHeight() / 2.0f + mBackgroundHeight / 2.0f;

    }

    /** 这个我也不想说
     * 如果三个月之后我看不懂这个代码
     * 说明
     * 我应该是失忆了
     * @return
     */
    public int getProgress() {
        return mCurrentProgress;
    }

    /**
     * 同上
     * @return
     */
    public int getMax() {
        return mMaxProgress;
    }

    /**
     * 同上
     * @param progress
     */
    public void setProgress(int progress) {
        mCurrentProgress = progress;
        invalidate();
    }

    /**
     * 让我的 progress 动起来！
     * ValueAnimator 是个神器
     * Google一下
     * @param progress
     */
    public void anitProgress(int progress) {
        ValueAnimator animator = ValueAnimator.ofInt(0, progress);
        animator.setDuration(1500);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                mCurrentProgress = (int) animation.getAnimatedValue();
                postInvalidate();
            }
        });

        animator.start();
    }

    public float dp2px(float dp) {
        final float scale = getResources().getDisplayMetrics().density;
        return dp * scale + 0.5f;
    }
```

这就是我第一个实质意义上的自定义View了，以前一般都是扩展TextView啊，组合LinearLayout
之类的。
虽然这个View也是看着别人的代码写的。
不过总算是自己动手敲出来了。
敲开心啊。
我 ~~照抄~~学习的 project [代码家](https://github.com/daimajia)的[NumberProgressBar](https://github.com/daimajia/NumberProgressBar)
