---
title: 我的第二个自定义View
date: 2016-07-14 15:19:05
tags: Android
---
自古以来，第一个总是抢走了所有的聚光灯。
怎么办，我是第二个。

据说有一个国际惯例，要真相
那就来吧
![真相](http://7xv11f.com1.z0.glb.clouddn.com/my-second-custom-view.gif)

看到了吗。
下边那个圆环就是我们第二个主角啦。
上面那个是 传说中的第一个。
没办法，第二个不管再怎么好看也比不上 号称第一的那位。
这，应该是一个诅咒。
```

/**
 * Created by Jimbray  .
 * on 2016/7/13
 * Email: jimbray16@gmail.com
 * Description: 这是我第二个自定义View。
 * 第一个永远是被人铭记的，可是第二个呢
 * 问你一个问题：世界上最高的山峰是？
 * 答：当然是珠穆朗玛峰啦！
 * 那第二高的呢？
 * 答：当然是乔戈里峰啦！
 * -_- 你不按套路出牌啊，套路里你是不知道的！
 */
public class MySecondProgressbar extends View {

    private Context mContext;

    // 准备好画圆环的矩形区域
    private RectF mRectF = new RectF(0, 0, 0, 0);
    // 准备好画文字的矩形区域
    private RectF mTextRrectF = new RectF(0, 0, 0, 0);

    // 握好画笔啊，有三个频段，还有一个文字
    private Paint mHeavyProgressPaint, mModerateProgressPaint, mLightProgressPaint, mTextPaint;

    // 默认值设置
    private final float default_height  = dp2px(45);
    private final float default_width   = dp2px(45);
    private final int default_heavy_color = Color.rgb(255, 54, 102);
    private final int default_moderate_color = Color.rgb(255, 128, 97);
    private final int default_light_color = Color.rgb(8, 176, 242);
    private final int default_text_color = Color.rgb(0, 0, 0);
    private final float default_text_size = sp2px(14);
    private final float default_heavy_progress_width = dp2px(12);
    private final float defaule_moderate_progress_width = dp2px(12);
    private final float default_light_progress_width = dp2px(12);

    //三个频段，三种颜色啊，还有一个是字体颜色
    private int mHeavyColor, mModerateColor, mLightColor, mTextColor;
    //画的东西宽高，还是给个默认值好了
    private float mViewHeight, mViewWidth;
    //我们画的是圆环，要多粗有多粗，定海神针，啊不，定海神环
    private float mHeavyProgressWidth, mModerateProgressWidth, mLightProgresswidth;

    private boolean mIfDrawText; //是不是要画一下中间的文字呢。还是 写
    private String mCurrentDrawText; //画的文字你得告诉我吧
    private float mTextSize; //字别太大，会撑着
    private int mTextBaseLine; //这个 我是抄的

    private float mCircleOffset = dp2px(15); //人与人之间的距离不要太近，还是要有一定的安全距离。不然亲上了怎么办。

    private float mCurrentProgress; //这第二个自定义View还是一个progressbar
    private float mCurHeavy, mCurModerate, mCurLight; //三个频段，三个单词是突然想到的，不知道什么意思

    public enum MySecondProgressTextVisibility {// 跟第一个自定义View最大的不同就是 ：名字
        Visible, Invisible
    }

    public MySecondProgressbar(Context context) {
        super(context);
        init(context, null, 0);
    }

    public MySecondProgressbar(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs, 0);
    }

    public MySecondProgressbar(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs, defStyleAttr);
    }

    /**
     *
     * 不想解释
     * @param context
     * @param attrs
     * @param defStyleAttr
     */
    private void init(Context context, AttributeSet attrs, int defStyleAttr) {
        this.mContext = context;

        mHeavyColor = default_heavy_color;
        mModerateColor = default_moderate_color;
        mLightColor = default_light_color;
        mTextColor = default_text_color;

        mViewWidth = default_width;
        mViewHeight = default_height;

        mTextSize = default_text_size;

        mHeavyProgressWidth = default_heavy_progress_width;
        mModerateProgressWidth = defaule_moderate_progress_width;
        mLightProgresswidth = default_light_progress_width;

        mIfDrawText = true;

        initPainters();
    }

    /**
     * 上颜料
     */
    private void initPainters() {
        mHeavyProgressPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mHeavyProgressPaint.setColor(mHeavyColor);
        mHeavyProgressPaint.setStyle(Paint.Style.STROKE);
        mHeavyProgressPaint.setStrokeWidth(mHeavyProgressWidth);

        mModerateProgressPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mModerateProgressPaint.setColor(mModerateColor);
        mModerateProgressPaint.setStyle(Paint.Style.STROKE);
        mModerateProgressPaint.setStrokeWidth(mModerateProgressWidth);

        mLightProgressPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mLightProgressPaint.setColor(mLightColor);
        mLightProgressPaint.setStyle(Paint.Style.STROKE);
        mLightProgressPaint.setStrokeWidth(mLightProgresswidth);

        mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mTextPaint.setColor(mTextColor);
        mTextPaint.setTextAlign(Paint.Align.CENTER);
        mTextPaint.setTextSize(mTextSize);
    }

    /**
     * 最小宽度 由你定
     * 不定我来给个最小值吧
     * @return
     */
    @Override
    protected int getSuggestedMinimumWidth() {
        return (int) mViewWidth;
    }

    /**
     * 跟宽度一个意思
     * @return
     */
    @Override
    protected int getSuggestedMinimumHeight() {
        return (int) mViewHeight;
    }

    /**
     * @param canvas
     * 咱先分析一下
     * 要画什么
     * 我要画一个圆环
     * 圆环里面有三个频段的显示（不同颜色）
     * 这三个频段呢，分别代表三个数值的占比关系
     * 中间放个文字吧，不然太空
     * 好像就这样
     */
    @Override
    protected void onDraw(Canvas canvas) {
        if(mIfDrawText) {
            calWithText();
        } else {
            calWithoutText();
        }

        //上面已经把我们要画的区域圈出来了
        //可以施展拳脚，不对，是画笔了

        float heavy_sweep_angle = mCurHeavy / mCurrentProgress; //算一下占比
        heavy_sweep_angle = heavy_sweep_angle * 360; //占比最终还是要转换成角度的，0.5的话就是180度
        if(mCurHeavy != 0) {
            //从上面12点钟的位置开始，就是270啦，我不喜欢负数，扫过多大的角度呢
            canvas.drawArc(mRectF, 270, heavy_sweep_angle, false, mHeavyProgressPaint); //试试把false改为true玩一玩，顺便把paint 的stroke属性去掉。有惊喜哦！
        }

        //跟上面的意思差不多啦。
        float moderate_sweep_angle = mCurModerate / mCurrentProgress;
        moderate_sweep_angle = moderate_sweep_angle * 360;
        if(mCurModerate != 0) {
            canvas.drawArc(mRectF, 270 + heavy_sweep_angle, moderate_sweep_angle, false, mModerateProgressPaint);
        }

        //还是一样的。
        float light_sweep_angle = mCurLight / mCurrentProgress;
        light_sweep_angle = light_sweep_angle * 360;
        if(mCurLight != 0) {
            canvas.drawArc(mRectF, 270 + heavy_sweep_angle + moderate_sweep_angle, light_sweep_angle, false, mLightProgressPaint);
        }
        //上面已经把圆环都画完了

        //画个原点，我是来确定我写的字是不是在中间，不能我说在中间就是在中间是吧。如果它不是在中间，我把它说成在中间，然后被你发现它不在中间，那我不是很尴尬。
//        canvas.drawOval(getWidth() /2 - 1 , getHeight() / 2 - 1, getWidth() / 2 + 1, getHeight() / 2 + 1, mHeavyProgressPaint); //画个圆点，看看文字是不是居中的

        if(mIfDrawText) { //准备书法

//            canvas.drawRect(mTextRrectF, mLightProgressPaint); //画个矩形 把text 框起来 ,参照物
            //第一种
            //这个是不对的做法，我以为是写在了中间
//            canvas.drawText(mCurrentDrawText, getWidth() / 2.0f, getHeight() / 2.0f + (mTextPaint.descent() + mTextPaint.ascent()), mTextPaint);

            //第二种
            // 后来我知道我错了，所以我就将错就错，再错一次
//            float text_width = mTextPaint.measureText(mCurrentDrawText);
//            canvas.drawText(mCurrentDrawText, mTextRrectF.left + text_width/2.0f, mTextRrectF.bottom, mTextPaint);

            //第三种
            //又来了!快看我找到的解决方案：[等灯等灯](http://blog.csdn.net/hursing/article/details/18703599)
            canvas.drawText(mCurrentDrawText, mTextRrectF.centerX(), mTextBaseLine, mTextPaint);
        }

    }

    /**
     * 精华1
     * 这里不写字
     */
    private void calWithoutText() {
        mRectF.left = getPaddingLeft() + mCircleOffset;
        mRectF.top = getPaddingTop() + mCircleOffset;
        mRectF.right = getWidth() - getPaddingRight() - mCircleOffset;
        mRectF.bottom = getHeight() - getPaddingBottom() - mCircleOffset;

        //就完啦？
        //你说的对呀
        //我就确定个区域，咋滴啦
    }

    /**
     * 精华2
     */
    private void calWithText() {
        mRectF.left = getPaddingLeft() + mCircleOffset;
        mRectF.top = getPaddingTop() + mCircleOffset;
        mRectF.right = getWidth() - getPaddingRight() - mCircleOffset;
        mRectF.bottom = getHeight() - getPaddingBottom() - mCircleOffset;

        if(!TextUtils.isEmpty(mCurrentDrawText)) {
            float text_width = mTextPaint.measureText(mCurrentDrawText);
            mTextRrectF.left = getWidth() / 2.0f - text_width / 2.0f;
            mTextRrectF.top = getHeight() / 2.0f - mTextSize / 2.0f;
            mTextRrectF.right = getWidth() / 2.0f + text_width / 2.0f;
            mTextRrectF.bottom = getHeight() / 2.0f + mTextSize / 2.0f;

            //这里是计算 文字居中的解决方案 具体请访问我搜到的 [这是我找到的](http://blog.csdn.net/hursing/article/details/18703599)
            Paint.FontMetricsInt fontMetrics = mTextPaint.getFontMetricsInt();
            mTextBaseLine = (int) ((mTextRrectF.bottom + mTextRrectF.top - fontMetrics.bottom - fontMetrics.top) / 2);
        } else {
            mIfDrawText = false;
        }

        //这里还是确定个区域的问题
        //只不过多了文字的区域
    }

    /**
     * 这个套路还是可以再深一点
     * 不过我是新司机
     * 不会换档
     * @param widthMeasureSpec
     * @param heightMeasureSpec
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
        int padding = isWidth ? getPaddingLeft() + getPaddingRight() : getPaddingTop() + getPaddingBottom();
        if (mode == MeasureSpec.EXACTLY) {
            result = size;
        } else {
            result = isWidth ? getSuggestedMinimumWidth() : getSuggestedMinimumHeight();
            result += padding;
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


    public float dp2px(float dp) {
        final float scale = getResources().getDisplayMetrics().density;
        return dp * scale + 0.5f;
    }

    public float sp2px(float sp) {
        final float scale = getResources().getDisplayMetrics().scaledDensity;
        return sp * scale;
    }

    private void setProgressTextVisibility(MySecondProgressTextVisibility visibility) {
        mIfDrawText = visibility == MySecondProgressTextVisibility.Visible;
    }

    public void setProgress(float heavy, float moderate, float light) {
        this.mCurrentProgress = heavy + moderate + light;
        this.mCurHeavy = heavy;
        this.mCurModerate = moderate;
        this.mCurLight = light;
        invalidate();
    }

    /**
     * 加个动画
     * 就会变成屌屌的
     * 我知道我为什么选progressbar来做第一个自定义View了
     * 因为
     * 其他的我也不会啊
     * @param heavy
     * @param moderate
     * @param light
     */
    public void animProgress(float heavy, float moderate, float light) {
        this.mCurrentProgress = heavy + moderate + light;

        ValueAnimator heavy_animator = ValueAnimator.ofFloat(0, heavy);
        heavy_animator.setInterpolator(new LinearInterpolator());
        heavy_animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                mCurHeavy = (float) animation.getAnimatedValue();
                postInvalidate();
            }
        });

        ValueAnimator moderate_animator = ValueAnimator.ofFloat(0, moderate);
        moderate_animator.setInterpolator(new LinearInterpolator());
        moderate_animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                mCurModerate = (float) animation.getAnimatedValue();
                postInvalidate();
            }
        });


        ValueAnimator light_animator = ValueAnimator.ofFloat(0, light);
        light_animator.setInterpolator(new LinearInterpolator());
        light_animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                mCurLight = (float) animation.getAnimatedValue();
                postInvalidate();
            }
        });

        AnimatorSet animSet = new AnimatorSet();
        animSet.setDuration(1000);
        animSet.playTogether(light_animator, moderate_animator, heavy_animator);
        animSet.start();
    }

    public void setProgressText(String text) {
        this.mCurrentDrawText = text;
        invalidate();
    }
}

```

第二个也写完了。
这个我没有抄别人的啊。
我抄的是第一个自定义View 的。
敲完了。