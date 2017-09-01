---
layout: post
title: "关于getHeight与getMeasureHeight的问题"
description: "view"
category: height
tags: [width，height]
---

上上一篇讲了一些获取View宽高的一些方式，但这里会有一个问题可能有些人不知道，就是有些地方使用了getHeight，有些地方使用了getMeasureHeight，什么时候getHeight是有效的，什么时候getMeasureHeight是有效的，很多人都不是很清楚，大部分人只知道getMeasureHeight是获取一个View的测量高度，而getHeight是获取View的真实高度，那么问题来了，实时真的是如此吗？先看一个例子。

       <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="30dp"
        android:background="@color/colorAccent">
        <View
            android:id="@+id/text"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="@color/colorPrimaryDark"/>
    </LinearLayout>
  
父布局30dp高度，而内部的TextView的高度100dp已经超过了父布局的高度，这时候通过上一篇的方式获取View的高度是多少呢。结果如下。

12-22 20:41:57.511 3943-3943/com.example.zhangtao21.measureheight E/measure text1: 0
12-22 20:41:57.634 3943-3943/com.example.zhangtao21.measureheight E/global text1: 100
12-22 20:41:57.686 3943-3943/com.example.zhangtao21.measureheight E/runnable text1: 100
12-22 20:41:57.791 3943-3943/com.example.zhangtao21.measureheight E/onWindowFocusChanged :     text1:  100dp

除了直接以measure的方式获取到的高度是0外，其他的高度都是100dp，但是这个View的高度并没有100dp，那么说明getHeight并不一定获取到的是View的真实高度。关于getHeight和getMeasureHeight请记住这么一句话。getMeasureHeight是由自身决定的，getHeight是由他的父布局决定的。先解释一下getMeasureHeight就是自身的测量高度，不考虑父布局高度，我测量自己是多高拿到的getMeasureHeight就是多少，getHeight是父布局告诉我多高，我拿到的就是多少。两种方法获取到的都不是View的真实高度。为什么这么说请看源码。。。

View的getMeasureHeight：

    public final int getMeasuredHeight() {
        return mMeasuredHeight & MEASURED_SIZE_MASK;
    }

获取到的是View中的mMeasuredHeight的变量。这个变量的赋值是在onMeasure中调用的setMeasuredDimension(width, height)，我们平时在自定义View的时候在onMeasure中要让我们计算的宽高生效必须调用的方法。
setMeasuredDimensionRaw

    private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
        mMeasuredWidth = measuredWidth;
        mMeasuredHeight = measuredHeight;

        mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
    }

该方法会对 mMeasuredWidth，mMeasuredHeight进行赋值。setMeasuredDimension是final的，setMeasuredDimensionRaw是private，都不允许我们进行复写。onMeasure是有measure方法
调用。整个视图树的measure的过程如下。

![p](/img/getheight/measure.png)

而只要一个View调用了measure方法，获取到的getMeasureHeight就不是0，至于准不准确我们只能说呵呵了，在onMeasure中计算的根据不同的Mode获取到的数值会不一样，在什么情况下（Mode）这个View本身会有多高，全部要看onMeasure的实现了，而每一次调用Measure都会对mMeasuredHeight进行赋值，所以每一次measure后获取到的数值就有可能不同。

关于getheight的其实很简单，底部减去顶部。

    @ViewDebug.ExportedProperty(category = "layout")
    public final int getHeight() {
        return mBottom - mTop;
    }

赋值是在View的layout中

    @SuppressWarnings({"unchecked"})
    public void layout(int l, int t, int r, int b) {
     //****
        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
     //****
    }

只看这么一行 setOpticalFrame最终也会调用setFrame，在 setFrame中会对mLeft ，mTop ，mRight，mBottom 进行赋值，layout方法是由ViewGroup调用，所以mBottom和mTop最终都是由父布局决定的，父布局分配给子View多少获取到的就是多少。父布局在自己的onLayout中获取到每一个View的MeasureHeight，再依据MeasureHeight分配每一个布局的Height，Measureheight只是告诉父布局，子View想要多少，而最终分配了多少还是由父布局决定，既然我们知道了这个原理，我们写这样的一个ViewGroup，

    public class CustomViewGroup extends FrameLayout{
        public CustomViewGroup(Context context) {
            super(context);
        }

        public CustomViewGroup(Context context, AttributeSet attrs) {
            super(context, attrs);
        }

        public CustomViewGroup(Context context, AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
        }

        @Override
        protected void onLayout(boolean changed, int l, int t, int r, int b) {
            View view = getChildAt(0);
            view.layout(l,t,r,l+DensityUtil.dp2px(getContext(),300));
        }
    }

这个布局不会参考子布局的高度也不会考虑自身的高度，无论申请多少，都会给子View分配定高300dp，这样无论子View真实有多高，测量有多高，调用getHeight获取到的都是300dp，这么说来getheight获取到的并不是准确的，但这里我们的ViewGroup是不符合逻辑，不遵守任何约定的，使用常用的几大布局获取到的getHeight还是比较准确的，至于最开始的问题，想获取到我们可见的View高度，也有办法。

     view.post(new Runnable() {
                @Override
                public void run() {
                    int height = 0;
                    ViewGroup viewGroup = (ViewGroup) view.getParent();
                    if(view.getBottom() > viewGroup.getBottom()){
                         height = viewGroup.getBottom() - view.getTop();
                    } else {
                        height = view.getHeight();
                    }
                    Log.e("realheight"," "+DensityUtil.px2dp(Test1Activity.this, height));
                }
            });

如果子View没有EXACTLY方式超出父布局这是获取到的高度是准确的，如果超出了父布局，这是我们使用父布局的底部减去View的顶部，获取到的就是view的高度。

总结一下
前提正常情况下（无特殊赋值操作）：
在onMeasure调用了setMeasuredDimension后获取到的getMeasureHeight就不是0了。
在layout调用后，在onLayout中获取到的getHeight就不是0了，之后也不是0了。

在最后一次onMeasure后，getMeasureHeight获取到的数值就不会在改变了。
在最后一次layout调用后，在onLayout获取到的和之后获取到的getHeight也就不会变了，也就是正常情况下的准确数值。


