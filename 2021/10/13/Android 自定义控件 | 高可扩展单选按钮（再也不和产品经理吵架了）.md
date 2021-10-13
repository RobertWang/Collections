> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903778550824968)

这是该系列文章的第一篇，系列目录如下：

1.  [Android 自定义控件 | 高可扩展单选按钮（再也不和产品经理吵架了）](https://juejin.im/post/6844903778550824968 "https://juejin.im/post/6844903778550824968")
2.  [Android 自定义控件 | 运用策略模式扩展单选按钮和产品经理成为好朋友](https://juejin.im/post/6844903847056375815 "https://juejin.im/post/6844903847056375815")

业务场景
----

兴高采烈地前去一周一次的需求大会。为了更加精准的推送，需要采集用户信息，于是乎产品设计了如下界面：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/18/168fed05f1a021d8~tplv-t2oaga2asx-watermark.awebp)

没想到，在发版本的前一天，突然觉得采集粒度不够细，希望将 4 个选项增加为 6 个。面对这突如其来，猝不及防的需求变化，设计和研发组都极力反对。

对于设计来说，不仅仅是加两张图，若沿用之前的布局设计，屏幕就放不下 6 个选项，所以需要重新设计布局。经过设计小姐姐的加班努力，最终设计图改成这样： ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/18/168fed05f9c58825~tplv-t2oaga2asx-watermark.awebp)

对于开发来说。。。 单选按钮有两个标题？ 两个标题还是不同颜色？ 选中之后标题居然要变颜色？ 不怕不怕，别说明天就要发版本，就是今天晚上发也可以。因为我自定义了一个单选控件，这次界面的改动，只需要换 2 个布局文件。（**公司鼓励拥抱变化的价值观，对于开发来说写出 “拥抱变化” 的代码就是最好的回应**）

如何定义单选按钮这个抽象？
-------------

在原生抽象中，单选控件包含两个概念：

1.  **单选组**`RadioGroup`
2.  **单选按钮**`RadioButton`

**原生抽象的局限性在于**：`RadioGroup`和`RadioButton`是父子关系，即 RadioGroup 必须是一个明确的`ViewGroup`类型，这样就约束了 RadioButton 的布局方式。

**如果单选组不是一个`View`，是不是就可以解放这层约束？**

对于这个问题的答案留一个悬念，抛开单选组，先来看看单选按钮是一个怎么样的抽象。

单选按钮应该包含如下基本特性：

1.  是一个 View，且可点击
2.  有两种状态（选中、未选中），且对应不同的视图

只需要继承 View，并利用`View.isSelected()`就能实现这两个特性。代码如下：

```
public abstract class Selector extends FrameLayout implements View.OnClickListener {

    public Selector(Context context) {
        super(context);
        initView(context, null);
    }

    public Selector(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context, attrs);
    }

    public Selector(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    private void initView(Context context, AttributeSet attrs) {
        //实现特性1：可点击
        this.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        //实现特性2：点击后改变选中状态
        boolean isSelect = switchSelector();
    }

    //反转选中状态
    public boolean switchSelector() {
        boolean isSelect = this.isSelected();
        this.setSelected(!isSelect);
        return !isSelect;
    }
}
复制代码

```

为满足业务场景，需要新增附加特性：可自定义按钮内元素相对布局

附加特性会随着业务需求变化而变化，可以用`模版方法模式`将这层变化封装起来：由`Selector`定义初始化算法框架，将真正界面初始化延后到子类进行。

*   虽然这次业务场景中，单选按钮元素的布局是：图片在上，文字在下。下次换了咋办？所以定义元素布局应该作为一个抽象函数交给`Selector`子类实现。
*   为了实现选中的渐变效果，`Selector`需提供选项变更的时机。
*   按钮包含一些基本的属性，比如按钮名称，按钮图标，将这些属性写成自定义属性并传递给子类解析，代码如下：

```
public abstract class Selector extends FrameLayout implements View.OnClickListener {

    public Selector(Context context) {
        super(context);
        initView(context, null);
    }

    public Selector(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context, attrs);
    }

    public Selector(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    //模版方法
    private void initView(Context context, AttributeSet attrs) {
        //读取自定义属性
        if (attrs != null) {
            TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.Selector);
            int tagResId = typedArray.getResourceId(R.styleable.Selector_tag, 0);
            tag = context.getString(tagResId);
            //将自定义属性传递给子类
            onObtainAttrs(typedArray);
            typedArray.recycle();
        } else {
            tag = “default tag”;
        }
        //构建按钮视图
        View view = onCreateView();
        LayoutParams params = new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
        this.addView(view, params);
        this.setOnClickListener(this);
    }
    
    public void onObtainAttrs(TypedArray typedArray) {}

    //子类实现该函数以定义单选按钮元素布局
    protected abstract View onCreateView();

    @Override
    public void onClick(View v) {
        boolean isSelect = switchSelector();
    }

    public boolean switchSelector() {
        boolean isSelect = this.isSelected();
        this.setSelected(!isSelect);
        //当选项变更时
        onSwitchSelected(!isSelect);
        return !isSelect;
    }

    //选项变更
    protected abstract void onSwitchSelected(boolean isSelect);
}
复制代码

```

因为`Selector`是抽象类，所以必须由子类实现它的抽象，下面的代码即是 demo 中年龄单选按钮的实现：

```
public class AgeSelector extends Selector {
    //声明按钮包含的控件
    private TextView tvTitle;
    private ImageView ivIcon;
    private ImageView ivSelector;
    private ValueAnimator valueAnimator;
    
    @Override
    public void onObtainAttrs(TypedArray typedArray) {
        //解析自定义属性
        text = typedArray.getString(R.styleable.Selector_text);
        iconResId = typedArray.getResourceId(R.styleable.Selector_img, 0);
        indicatorResId = typedArray.getResourceId(R.styleable.Selector_indicator, 0);
        textColor = typedArray.getColor(R.styleable.Selector_text_color, Color.parseColor("#FF222222"));
        textSize = typedArray.getInteger(R.styleable.Selector_text_size, 15);
    }

    private void onBindView(String text, int iconResId, int indicatorResId, int textColor, int textSize) {
        //将自定义属性绑定到控件
        if (tvTitle != null) {
            tvTitle.setText(text);
            tvTitle.setTextSize(TypedValue.COMPLEX_UNIT_SP, textSize);
            tvTitle.setTextColor(textColor);
        }
        if (ivIcon != null) {
            ivIcon.setImageResource(iconResId);
        }
        if (ivSelector != null) {
            ivSelector.setImageResource(indicatorResId);
            ivSelector.setAlpha(0);
        }
    }

    @Override
    protected View onCreateView() {
        //构建自定义按钮布局
        View view = LayoutInflater.from(this.getContext()).inflate(R.layout.age_selector, null);
        tvTitle = view.findViewById(R.id.tv_title);
        ivIcon = view.findViewById(R.id.iv_icon);
        ivSelector = view.findViewById(R.id.iv_selector);
        onBindView(text, iconResId, indicatorResId, textColor, textSize);
        return view;
    }

    @Override
    protected void onSwitchSelected(boolean isSelect) {
        //单选按钮状态变化时做动画
        if (isSelect) {
            playSelectedAnimation();
        } else {
            playUnselectedAnimation();
        }
    }

    private void playUnselectedAnimation() {
        if (ivSelector == null) {
            return;
        }
        if (valueAnimator != null) {
            valueAnimator.reverse();
        }
    }

    private void playSelectedAnimation() {
        if (ivSelector == null) {
            return;
        }
        valueAnimator = ValueAnimator.ofInt(0, 255);
        valueAnimator.setDuration(800);
        valueAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                ivSelector.setAlpha((int) animation.getAnimatedValue());
            }
        });
        valueAnimator.start();
    }
}
复制代码

```

其中单选按钮的布局文件如下：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/iv_selector"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@id/tv_title"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_chainStyle="spread"
        app:layout_constraintVertical_weight="122" />

    <ImageView
        android:id="@+id/iv_icon"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintDimensionRatio="1:1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.026"
        app:layout_constraintWidth_percent=".81" />

    <TextView
        android:id="@+id/tv_title"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:gravity="center_horizontal|bottom"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@id/iv_selector"
        app:layout_constraintVertical_chainStyle="spread"
        app:layout_constraintVertical_weight="28" />
</android.support.constraint.ConstraintLayout>
复制代码

```

如何定义单选组这个抽象？
------------

等等，好像有点不太对劲！如果运行上述代码，你会发现每个`Selector`都运行良好（选中状态发生变化时有渐变动画），但多个`Selector`可以同时被选中，他们并没有实现互斥选中。。。

定神一想，发现原因是`Selector`这个抽象只关心自己的选中状态，它并不知道其他`Selector`的状态。

所以原生控件需要`RadioGroup`这个角色，它作为父亲，了解每个孩子的动向！

但我们不想要一个`ViewGroup`类型的父亲，因为它管的太多，孩子不能随意布局，局限性大。

那就造一个看不见的父亲！其实父亲做的事情不就是 **“在一个孩子选中的时候，通知另一个孩子取消选中” 吗？**

有了思路动手就干，代码如下：

```
public class SelectorGroup {
    //用于保存上次选中按钮
    private HashMap<String, Selector> selectorMap = new HashMap<>();

    //获取上次选中按钮
    public Selector getPreSelector(String groupTag) {
        return selectorMap.get(groupTag);
    }

    //取消上次选中按钮的选中状态
    private void cancelPreSelector(Selector selector) {
        String groupTag = selector.getGroupTag();
        Selector preSelector = getPreSelector(groupTag);
        if (preSelector != null) {
            preSelector.setSelected(false);
        }
    }
    
    //当单选组按钮被点击时，点击事件会通过这个方法传递给单选组
    void onSelectorClick(Selector selector) {
        selector.setSelected(true);
        cancelPreSelector(selector);
        //将这次选中按钮保存在map中
        selectorMap.put(selector.getGroupTag(), selector);
    }
}
复制代码

```

为了让`SelectorGroup`统一管理按钮点击后的选中和取消状态变更，需要将按钮的点击事件传递给它，遂修改单选按钮代码如下：

```
public abstract class Selector extends FrameLayout implements View.OnClickListener {
    @Override
    public void onClick(View v) {
        //传递点击事件给SelectorGroup，它会调用setSelected()来统一管理按钮选中和取消状态
        if (selectorGroup != null) {
            selectorGroup.onSelectorClick(this);
        }
    }
    
    @Override
    public void setSelected(boolean selected) {
        boolean isPreSelected = isSelected();
        super.setSelected(selected);
        if (isPreSelected != selected) {
            onSwitchSelected(selected);
        }
    }

    public boolean switchSelector() {
        boolean isSelect = this.isSelected();
        this.setSelected(!isSelect);
        //当选项变更时
        onSwitchSelected(!isSelect);
        return !isSelect;
    }

    //选项变更
    protected abstract void onSwitchSelected(boolean isSelect);
}
复制代码

```

现在就可以像这样使用自定义单选按钮了：

```
public class MainActivity extends AppCompatActivity{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
    }

    private void initView() {
        SelectorGroup singleGroup = new SelectorGroup();
        singleGroup.setChoiceMode(SelectorGroup.MODE_SINGLE_CHOICE);
        singleGroup.setStateListener(new SingleChoiceListener());
        ((Selector) findViewById(R.id.single10)).setGroup("single", singleGroup);
        ((Selector) findViewById(R.id.single20)).setGroup("single", singleGroup);
        ((Selector) findViewById(R.id.single30)).setGroup("single", singleGroup);
    }

    private class SingleChoiceListener implements SelectorGroup.StateListener {

        @Override
        public void onStateChange(String tag, boolean isSelected) {
            Toast.makeText(MainActivity.this, tag + " is selected", Toast.LENGTH_SHORT).show();
        }
    }
}
复制代码

```

更多
--

除了能快速响应需求变化外，`Selector`还可以实现更多自定义效果。如下图是个三选一单选组件，选项分居两行形成三角形，且带有渐变选中效果。 ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/18/168fed05fa4c8a5b~tplv-t2oaga2asx-watermark.awebp)

*   相比较而言，原生控件`RadioButton`有如下的局限性：
    1.  不能自定义按钮选中动画效果
    2.  不能自定义按钮相对布局。

`RadioGroup`继承自`LinearLayout`，所以`RadioButton`的排列方式只能是横向或纵向一字排开。

*   用本文中的`Selector`就可以轻而易举的实现这个效果。

talk is cheap ,[show me the code](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwisdomtl%2FSelector "https://github.com/wisdomtl/Selector")
------------------------------------------------------------------------------------------------------------------------------------------------------

推荐阅读
----

[这样写代码就能和产品经理成为好朋友——策略模式实战](https://juejin.im/post/6844903847056375815 "https://juejin.im/post/6844903847056375815")