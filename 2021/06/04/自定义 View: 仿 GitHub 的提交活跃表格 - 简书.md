> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/15cfb35957b7)

### 说明

本文可能需要一些基础知识点，如 Canvas,Paint，Path，Rect 等类的基本使用，建议不熟悉的同学可以学习 [GcsSloop 安卓自定义 View 教程目录](https://link.jianshu.com?t=http://www.gcssloop.com/customview/CustomViewIndex)，会帮助很大。  

![](http://upload-images.jianshu.io/upload_images/1627943-dcce7359a3bbaf1a.jpg) github.jpg

上图就是 github 的提交表格，直观来看可以分为几个部分进行绘制：  
（1）各个月份的小方格子，并且色彩根据提交次数变化，由浅到深  
（2）右下边的颜色标志，我们右对齐就可以了  
（3）左边的星期，原图是从周日画到周六，我们从周一画到周日  
（4）上面的月份，我们只画出 1-12 月  
（5）点击时候弹出当天的提交情况，由一个小三角和圆角矩形组成  
需要解决的计算问题：  
（1）生成任意一年的所有天，包含年月日周，提交次数，色块颜色，坐标  
（1）一年中所有的小方格子坐标  
（2）右下边颜色标志坐标  
（3）左边星期坐标  
（4）上面月份坐标  
（5）点击弹出的提示框和文字坐标

### 生成某年所有天数

每天的信息我们需要封装成一个类，代码如下：

```
/**
 * Created by Administrator on 2017/1/13.
 * 封装每天的属性，方便在绘制的时候进行计算
 */
public class Day implements Serializable{
    /**年**/
    public int year;
    /**月**/
    public int month;
    /**日**/
    public int date;
    /**周几**/
    public int week;
    /**贡献次数，默认0**/
    public int contribution = 0;
    /**默认颜色,根据提交次数改变**/
    public int colour = 0xFFEEEEEE;

    /**方格坐标，左上点，右下点，确定矩形范围**/
    public float startX;
    public float startY;
    public float endX;
    public float endY;

    @Override
    public String toString() {
        //这里直接在弹出框中显示
        return ""+year+"年"+month+"月"+date+"日周"+week+","+contribution+"次";
    }
}


```

要想先绘制表格，需要计算出所有的天，这里计算一年中所有的天，我们通过从当年 1 月 1 日算起，到 12 月 31 日，因为星期是连续的，所以我们需要我们提供某年的 1 月 1 日是周几，比如 2016 年 1 月 1 日是周 5，这里必要的参数是 2016 和周 5，那么我们用一个类来实现该方法，代码如下：

```
public class DateFactory {

    /**平年map，对应月份和天数**/
    private static HashMap<Integer,Integer> monthMap = new LinkedHashMap<>(12);
    /**闰年map，对应月份和天数**/
    private static HashMap<Integer,Integer> leapMonthMap = new LinkedHashMap<>(12);

    static {
        //初始化map,只有2月份不同
        monthMap.put(1,31);leapMonthMap.put(1,31);
        monthMap.put(2,28);leapMonthMap.put(2,29);
        monthMap.put(3,31);leapMonthMap.put(3,31);
        monthMap.put(4,30);leapMonthMap.put(4,30);
        monthMap.put(5,31);leapMonthMap.put(5,31);
        monthMap.put(6,30);leapMonthMap.put(6,30);
        monthMap.put(7,31);leapMonthMap.put(7,31);
        monthMap.put(8,31);leapMonthMap.put(8,31);
        monthMap.put(9,30);leapMonthMap.put(9,30);
        monthMap.put(10,31);leapMonthMap.put(10,31);
        monthMap.put(11,30);leapMonthMap.put(11,30);
        monthMap.put(12,31);leapMonthMap.put(12,31);
    }

    /**
     * 输入年份和1月1日是周几
     * 闰年为366天,平年为365天
     * @param year    年份
     * @param weekday 该年1月1日为周几
     * @return 该年1月1日到12月31日所有的天数
     */
    public static List<Day> getDays(int year, int weekday) {
        List<Day> days = new ArrayList<>();
        boolean isLeapYear = isLeapYear(year);
        int dayNum = isLeapYear ? 366 : 365;
        Day day;
        int lastWeekday = weekday;
        for (int i = 1; i <= dayNum; i++) {
            day = new Day();
            day.year = year;
            //计算当天为周几,如果大于7就重置1
            day.week = lastWeekday<= 7 ? lastWeekday : 1;
            //计算当天为几月几号
            int[] monthAndDay = getMonthAndDay(isLeapYear, i);
            day.month = monthAndDay[0];
            day.date = monthAndDay[1];
            //记录下昨天是周几并+1
            lastWeekday = day.week;
            lastWeekday++;
            days.add(day);
        }
        checkDays(days);
        return days;
    }


    /**
     * 获取月和日
     * @param isLeapYear 是否闰年
     * @param currentDay 当前天数
     * @return 包含月和天的数组
     */
    public static int[] getMonthAndDay(boolean isLeapYear,int currentDay) {
        HashMap<Integer,Integer> maps = isLeapYear?leapMonthMap:monthMap;
        Set<Map.Entry<Integer,Integer>> set = maps.entrySet();
        int count = 0;
        Map.Entry<Integer, Integer> month = null;
        for (Map.Entry<Integer, Integer> entry : set) {
            count+=entry.getValue();
            if (currentDay<=count){
                month = entry;
                break;
            }
        }
        if (month == null){
            throw new IllegalStateException("未找到所在的月份");
        }
        int day = month.getValue()-(count-currentDay);
        return new int[]{month.getKey(),day};
    }


    /**
     * 判断是闰年还是平年
     * @param year 年份
     * @return true 为闰年
     */
    public static boolean isLeapYear(int year) {
        return year % 4 == 0 && year % 100 != 0 || year % 400 == 0;
    }

    /**
     * 检测生成的天数是否正常
     * @param days
     */
    private static void checkDays(List<Day> days) {
        if (days == null) {
            throw new IllegalArgumentException("天数为空");
        }
        if (days.size() != 365 && days.size() != 366) {
            throw new IllegalArgumentException("天数异常:" + days.size());
        }
    }

    public static void main(String[] args){
        //test
        List<Day> days = DateFactory.getDays(2016, 5);
        for (int i = 0; i < days.size(); i++) {
            System.out.println(days.get(i).toString());
        }
    }
}


```

具体的计算逻辑可以看看代码，不是很难，这样我们就能得到某年的所有天。

### 绘制天数格子

因为该 view 比较长，所以需要横屏显示，方便起见，这里我们也不再进行 view 的测量计算，也不再进行自定义属性，只关注其核心逻辑即可。  
首先我们需要将需要的成员变量定义出来：

```
    /**灰色方格的默认颜色**/
    private final static int DEFAULT_BOX_COLOUR = 0xFFEEEEEE;
    /**提交次数颜色值**/
    private final static int[] COLOUR_LEVEL =
            new int[]{0xFF1E6823, 0xFF44A340, 0xFF8CC665, 0xFFD6E685, DEFAULT_BOX_COLOUR};
    /**星期**/
    private String[] weeks = new String[]{"Mon", "Wed", "Fri", "Sun"};
    /**月份**/
    private String[] months =
            new String[]{"Jan", "Feb", "Mar", "Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"};
    /**默认的padding,绘制的时候不贴边画**/
    private int padding = 24;
    /**小方格的默认边长**/
    private int boxSide = 8;
    /**小方格间的默认间隔**/
    private int boxInterval = 2;
    /**所有周的列数**/
    private int column = 0;

    private List<Day> mDays;//一年中所有的天
    private Paint boxPaint;//方格画笔
    private Paint textPaint;//文字画笔
    private Paint infoPaint;//弹出框画笔

    private Paint.FontMetrics metrics;//测量文字

    private float downX;//按下的点的X坐标
    private float downY;//按下的点的Y坐标
    private Day clickDay;//按下所对应的天


```

这些提取的变量是慢慢增加的，在自定义的时候一下想不全的时候可以先写，等用到某些变量的时候就提取出来。  
然后我们初始化一下数据：

```
    public GitHubContributionView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    public void initView() {
        mDays = DateFactory.getDays(2016, 5);
        //方格画笔
        boxPaint = new Paint();
        boxPaint.setStyle(Paint.Style.FILL);
        boxPaint.setStrokeWidth(2);
        boxPaint.setColor(DEFAULT_BOX_COLOUR);
        boxPaint.setAntiAlias(true);
        //文字画笔
        textPaint = new Paint();
        textPaint.setStyle(Paint.Style.FILL);
        textPaint.setColor(Color.GRAY);
        textPaint.setTextSize(12);
        textPaint.setAntiAlias(true);
        //弹出的方格信息画笔
        infoPaint = new Paint();
        infoPaint.setStyle(Paint.Style.FILL);
        infoPaint.setColor(0xCC888888);
        infoPaint.setTextSize(12);
        infoPaint.setAntiAlias(true);
        //将默认值转换px
        padding = UI.dp2px(getContext(), padding);
        boxSide = UI.dp2px(getContext(), boxSide);

        metrics = textPaint.getFontMetrics();
    }


```

这里我们以 2016 年来举例，mDays 就是获取 2016 年的所有天的集合（参数可以当作自定义属性提取出来），相关的 Paint 也已经初始化好了，接下来就需要在 onDraw 方法里画，先画所有的方格子和月份标志:

```
    /**
     * 画出1-12月方格小块和上面的月份
     * @param canvas 画布
     */
    private void drawBox(Canvas canvas) {
        //方格的左上右下坐标
        float startX, startY, endX, endY;
        //起始月份为1月
        int month = 1;
        for (int i = 0; i < mDays.size(); i++) {
            Day day = mDays.get(i);
            if (i == 0){
                //画1月的文本标记,坐标应该是x=padding,y=padding-boxSide/2(间隙),y坐标在表格上面一点
                canvas.drawText(months[0],padding,padding-boxSide/2,textPaint);
            }
            if (day.week == 1 && i != 0) {
                //如果当天是周1，那么说明增加了一列
                column++;
                //如果列首的月份有变化，那么说明需要画月份
                if (day.month>month){
                    month = day.month;
                    //月份文本的坐标计算,x坐标在变化，而y坐标都是一样的，boxSide/2(间隙)
                    canvas.drawText(months[month-1],padding+column*(boxSide+boxInterval),padding-boxSide/2,textPaint);
                }
            }
            //计算方格坐标点,x坐标随列数的增多而增加,y坐标随行数的增多而变化
            startX = padding + column * (boxSide + boxInterval);
            startY = padding + (day.week - 1) * (boxSide + boxInterval);
            endX = startX + boxSide;
            endY = startY + boxSide;
            //将该方格的坐标保存下来,这样可以在点击方格的时候计算弹框的坐标
            day.startX = startX;
            day.startY = startY;
            day.endX = endX;
            day.endY = endY;
            //给画笔设置当前天的颜色
            boxPaint.setColor(day.colour);
            canvas.drawRect(startX, startY, endX, endY, boxPaint);
        }
        boxPaint.setColor(DEFAULT_BOX_COLOUR);//恢复默认颜色
    }


```

这里主要是注意下行数列数的变化和月份坐标的计算，格子画好了。

### 绘制星期文本

我们再画左边的星期文本：

```
 /**
     * 画左侧的星期
     * @param canvas 画布
     */
    private void drawWeek(Canvas canvas) {
        //文字是左对齐,所以找出最长的字
        float textLength = 0;
        for (String week : weeks) {
            float tempLength = textPaint.measureText(week);
            if (textLength < tempLength) {
                textLength = tempLength;
            }
        }
        //依次画出星期文本,坐标点x=padding-文本长度-文本和方格的间隙,y坐标随行数变化
        canvas.drawText(weeks[0], padding - textLength - 2, padding + boxSide - metrics.descent, textPaint);
        canvas.drawText(weeks[1], padding - textLength - 2, padding + 3 * (boxSide + boxInterval) - metrics.descent, textPaint);
        canvas.drawText(weeks[2], padding - textLength - 2, padding + 5 * (boxSide + boxInterval) - metrics.descent, textPaint);
        canvas.drawText(weeks[3], padding - textLength - 2, padding + 7 * (boxSide + boxInterval) - metrics.descent, textPaint);
    }


```

### 绘制颜色深浅标志

然后根据表格的高度再画出右下边的颜色深浅标志：

```
/**
     * 画出右下角的颜色深浅标志，因为是右对齐的所以需要从右往左画
     * @param canvas 画布
     */
    private void drawTag(Canvas canvas) {
        //首先计算出两个文本的长度
        float moreLength = textPaint.measureText("More");
        float lessLength = textPaint.measureText("Less");
        //画 More 文本,x坐标=padding+（列数+1）*（方格边长+方格间隙）-一个方格间隙-文本长度
        float moreX = padding + (column + 1) * (boxSide + boxInterval) - boxInterval - moreLength;
        //y坐标=padding+（方格行数+1,和表格底部有些距离）*（方格边长+方格间隙）+字体的ascent高度
        float moreY = padding + 8 * (boxSide + boxInterval) + Math.abs(metrics.ascent);
        canvas.drawText("More", moreX, moreY, textPaint);
        //画深浅色块,坐标根据上面的More依次计算就可以了
        float interval = boxSide - 2;//文字和色块间的距离
        float leftX = moreX - interval - boxSide;
        float topY = moreY - boxSide;
        float rightX = moreX - interval;
        float bottomY = moreY;//色块的Y坐标是一样的
        for (int i = 0; i < COLOUR_LEVEL.length; i++) {
            boxPaint.setColor(COLOUR_LEVEL[i]);
            canvas.drawRect(leftX - i * (boxSide + boxInterval), topY, rightX - i * (boxSide + boxInterval), bottomY, boxPaint);
        }
        //最后画 Less 文本,原理同上
        canvas.drawText("Less", leftX - 4 * (boxSide + boxInterval) - interval - lessLength, moreY, textPaint);
    }


```

这样整个表格主体绘制完成。

### 处理点击事件

接下来要处理点击事件，判断点击的坐标如果在方格内，那么弹出对于的文本框，先处理点击事件：

```
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //获取ACTION_DOWN的坐标，用来判断点在哪天，并弹出·
        if (MotionEvent.ACTION_DOWN == event.getAction()) {
            downX = event.getX();
            downY = event.getY();
            findClickBox();
        }
       //这里因为我们只是记录坐标点，不对事件进行拦截所以默认返回
        return super.onTouchEvent(event);
    }


```

判断是否在方格内：

```
    /**
     * 判断是否点击在方格内
     */
    private void findClickBox() {
        for (Day day : mDays) {
            //检测点击的坐标如果在方格内，则弹出信息提示
            if (downX >= day.startX && downX <= day.endX && downY >= day.startY && downY <= day.endY) {
                clickDay = day;//纪录点击的哪天
                break;
            }
        }
        //点击完要刷新，这样每次点击不同的方格，弹窗就可以在相应的位置显示
        refreshView();
    }

    /**
     * 点击弹出文字提示
     */
    private void refreshView() {
        invalidate();
    }


```

### 绘制弹出文本框

然后看看弹出文本框的绘制：

```
/**
     * 画方格上的文字弹框
     * @param canvas 画布
     */
    private void drawPopupInfo(Canvas canvas) {
        if (clickDay != null) {//点击的天不为null时候才画
            //先根据方格来画出一个小三角形，坐标就是方格的中间
            Path infoPath = new Path();
            //先从方格中心
            infoPath.moveTo(clickDay.startX + boxSide / 2, clickDay.startY + boxSide / 2);
            //然后是方格的左上点
            infoPath.lineTo(clickDay.startX, clickDay.startY);
            //然后是方格的右上点
            infoPath.lineTo(clickDay.endX, clickDay.startY);
            //画出三角
            canvas.drawPath(infoPath,infoPaint);
            //画三角上的圆角矩形
            textPaint.setColor(Color.WHITE);
            //得到当天的文本信息
            String popupInfo = clickDay.toString();
            System.out.println(popupInfo);
            //计算文本的高度和长度用以确定矩形的大小
            float infoHeight = metrics.descent - metrics.ascent;
            float infoLength = textPaint.measureText(popupInfo);
            Log.e("height",infoHeight+"");
            Log.e("length",infoLength+"");
            //矩形左上点应该是x=当前天的x+边长/2-（文本长度/2+文本和框的间隙）
            float leftX = (clickDay.startX + boxSide / 2 ) - (infoLength / 2 + boxSide);
            //矩形左上点应该是y=当前天的y+边长/2-（文本高度+上下文本和框的间隙）
            float topY = clickDay.startY-(infoHeight+2*boxSide);
            //矩形的右下点应该是x=leftX+文本长度+文字两边和矩形的间距
            float rightX = leftX+infoLength+2*boxSide;
            //矩形的右下点应该是y=当前天的y
            float bottomY = clickDay.startY;
            System.out.println(""+leftX+"/"+topY+"/"+rightX+"/"+bottomY);
            RectF rectF = new RectF(leftX, topY, rightX, bottomY);
            canvas.drawRoundRect(rectF,4,4,infoPaint);
            //绘制文字,x=leftX+文字和矩形间距,y=topY+文字和矩形上面间距+文字顶到基线高度
            canvas.drawText(popupInfo,leftX+boxSide,topY+boxSide+Math.abs(metrics.ascent),textPaint);
            clickDay = null;//重新置空，保证点击方格外信息消失
            textPaint.setColor(Color.GRAY);//恢复画笔颜色
        }
    }


```

这样主体逻辑完成，但需要开放设置某天提交次数的方法：

```
 /**
     * 设置某天的次数
     * @param year 年
     * @param month 月
     * @param day 日
     * @param contribution 次数
     */
    public void setData(int year,int month,int day,int contribution){
        //先找到是第几天，为了方便不做参数检测了
        for (Day d : mDays) {
            if (d.year == year && d.month == month && d.date == day){
                d.contribution = contribution;
                d.colour = getColour(contribution);
                break;
            }
        }
        refreshView();
    }

    /**
     * 根据提交次数来获取颜色值
     * @param contribution 提交的次数
     * @return 颜色值
     */
    private int getColour(int contribution){
        int colour = 0;
        if (contribution <= 0){
            colour = COLOUR_LEVEL[4];
        }
        if (contribution == 1){
            colour = COLOUR_LEVEL[3];
        }
        if (contribution == 2){
            colour = COLOUR_LEVEL[2];
        }
        if (contribution == 3){
            colour = COLOUR_LEVEL[1];
        }
        if (contribution >= 4){
            colour = COLOUR_LEVEL[0];
        }
        return colour;
    }



```

好了，所有逻辑完成，主要涉及到一些计算，完整代码：

```
/**
 * Created by Administrator on 2017/1/13.
 * 仿GitHub的提交活跃表
 * 横屏使用
 */

public class GitHubContributionView extends View {

    /**灰色方格的默认颜色**/
    private final static int DEFAULT_BOX_COLOUR = 0xFFEEEEEE;
    /**提交次数颜色值**/
    private final static int[] COLOUR_LEVEL =
            new int[]{0xFF1E6823, 0xFF44A340, 0xFF8CC665, 0xFFD6E685, DEFAULT_BOX_COLOUR};
    /**星期**/
    private String[] weeks = new String[]{"Mon", "Wed", "Fri", "Sun"};
    /**月份**/
    private String[] months =
            new String[]{"Jan", "Feb", "Mar", "Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"};
    /**默认的padding,绘制的时候不贴边画**/
    private int padding = 24;
    /**小方格的默认边长**/
    private int boxSide = 8;
    /**小方格间的默认间隔**/
    private int boxInterval = 2;
    /**所有周的列数**/
    private int column = 0;

    private List<Day> mDays;//一年中所有的天
    private Paint boxPaint;//方格画笔
    private Paint textPaint;//文字画笔
    private Paint infoPaint;//弹出框画笔

    private Paint.FontMetrics metrics;//测量文字

    private float downX;//按下的点的X坐标
    private float downY;//按下的点的Y坐标
    private Day clickDay;//按下所对应的天

    public GitHubContributionView(Context context) {
        this(context, null);
    }

    public GitHubContributionView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public GitHubContributionView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    public void initView() {
        mDays = DateFactory.getDays(2016, 5);
        //方格画笔
        boxPaint = new Paint();
        boxPaint.setStyle(Paint.Style.FILL);
        boxPaint.setStrokeWidth(2);
        boxPaint.setColor(DEFAULT_BOX_COLOUR);
        boxPaint.setAntiAlias(true);
        //文字画笔
        textPaint = new Paint();
        textPaint.setStyle(Paint.Style.FILL);
        textPaint.setColor(Color.GRAY);
        textPaint.setTextSize(12);
        textPaint.setAntiAlias(true);
        //弹出的方格信息画笔
        infoPaint = new Paint();
        infoPaint.setStyle(Paint.Style.FILL);
        infoPaint.setColor(0xCC888888);
        infoPaint.setTextSize(12);
        infoPaint.setAntiAlias(true);
        //将默认值转换px
        padding = UI.dp2px(getContext(), padding);
        boxSide = UI.dp2px(getContext(), boxSide);

        metrics = textPaint.getFontMetrics();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        column = 0;
        canvas.save();
        drawBox(canvas);
        drawWeek(canvas);
        drawTag(canvas);
        drawPopupInfo(canvas);
        canvas.restore();
    }

    /**
     * 画出1-12月方格小块和上面的月份
     * @param canvas 画布
     */
    private void drawBox(Canvas canvas) {
        //方格的左上右下坐标
        float startX, startY, endX, endY;
        //起始月份为1月
        int month = 1;
        for (int i = 0; i < mDays.size(); i++) {
            Day day = mDays.get(i);
            if (i == 0){
                //画1月的文本标记,坐标应该是x=padding,y=padding-boxSide/2(间隙),y坐标在表格上面一点
                canvas.drawText(months[0],padding,padding-boxSide/2,textPaint);
            }
            if (day.week == 1 && i != 0) {
                //如果当天是周1，那么说明增加了一列
                column++;
                //如果列首的月份有变化，那么说明需要画月份
                if (day.month>month){
                    month = day.month;
                    //月份文本的坐标计算,x坐标在变化，而y坐标都是一样的，boxSide/2(间隙)
                    canvas.drawText(months[month-1],padding+column*(boxSide+boxInterval),padding-boxSide/2,textPaint);
                }
            }
            //计算方格坐标点,x坐标一致随列数的增多而增加,y坐标随行数的增多而变化
            startX = padding + column * (boxSide + boxInterval);
            startY = padding + (day.week - 1) * (boxSide + boxInterval);
            endX = startX + boxSide;
            endY = startY + boxSide;
            //将该方格的坐标保存下来,这样可以在点击方格的时候计算弹框的坐标
            day.startX = startX;
            day.startY = startY;
            day.endX = endX;
            day.endY = endY;
            //给画笔设置当前天的颜色
            boxPaint.setColor(day.colour);
            canvas.drawRect(startX, startY, endX, endY, boxPaint);
        }
        boxPaint.setColor(DEFAULT_BOX_COLOUR);//恢复默认颜色
    }

    /**
     * 画左侧的星期
     * @param canvas 画布
     */
    private void drawWeek(Canvas canvas) {
        //文字是左对齐,所以找出最长的字
        float textLength = 0;
        for (String week : weeks) {
            float tempLength = textPaint.measureText(week);
            if (textLength < tempLength) {
                textLength = tempLength;
            }
        }
        //依次画出星期文本,坐标点x=padding-文本长度-文本和方格的间隙,y坐标随行数变化
        canvas.drawText(weeks[0], padding - textLength - 2, padding + boxSide - metrics.descent, textPaint);
        canvas.drawText(weeks[1], padding - textLength - 2, padding + 3 * (boxSide + boxInterval) - metrics.descent, textPaint);
        canvas.drawText(weeks[2], padding - textLength - 2, padding + 5 * (boxSide + boxInterval) - metrics.descent, textPaint);
        canvas.drawText(weeks[3], padding - textLength - 2, padding + 7 * (boxSide + boxInterval) - metrics.descent, textPaint);
    }

    /**
     * 画出右下角的颜色深浅标志，因为是右对齐的所以需要从右往左画
     * @param canvas 画布
     */
    private void drawTag(Canvas canvas) {
        //首先计算出两个文本的长度
        float moreLength = textPaint.measureText("More");
        float lessLength = textPaint.measureText("Less");
        //画 More 文本,x坐标=padding+（列数+1）*（方格边长+方格间隙）-一个方格间隙-文本长度
        float moreX = padding + (column + 1) * (boxSide + boxInterval) - boxInterval - moreLength;
        //y坐标=padding+（方格行数+1,和表格底部有些距离）*（方格边长+方格间隙）+字体的ascent高度
        float moreY = padding + 8 * (boxSide + boxInterval) + Math.abs(metrics.ascent);
        canvas.drawText("More", moreX, moreY, textPaint);
        //画深浅色块,坐标根据上面的More依次计算就可以了
        float interval = boxSide - 2;//文字和色块间的距离
        float leftX = moreX - interval - boxSide;
        float topY = moreY - boxSide;
        float rightX = moreX - interval;
        float bottomY = moreY;//色块的Y坐标是一样的
        for (int i = 0; i < COLOUR_LEVEL.length; i++) {
            boxPaint.setColor(COLOUR_LEVEL[i]);
            canvas.drawRect(leftX - i * (boxSide + boxInterval), topY, rightX - i * (boxSide + boxInterval), bottomY, boxPaint);
        }
        //最后画 Less 文本,原理同上
        canvas.drawText("Less", leftX - 4 * (boxSide + boxInterval) - interval - lessLength, moreY, textPaint);
    }


    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //获取点击时候的坐标，用来判断点在哪天，并弹出·
        if (MotionEvent.ACTION_DOWN == event.getAction()) {
            downX = event.getX();
            downY = event.getY();
            findClickBox();
        }
        return super.onTouchEvent(event);
    }

    /**
     * 判断是否点击在方格内
     */
    private void findClickBox() {
        for (Day day : mDays) {
            //检测点击的坐标如果在方格内，则弹出信息提示
            if (downX >= day.startX && downX <= day.endX && downY >= day.startY && downY <= day.endY) {
                clickDay = day;//纪录点击的哪天
                break;
            }
        }
        //点击完要刷新，这样每次点击不同的方格，弹窗就可以在相应的位置显示
        refreshView();
    }

    /**
     * 点击弹出文字提示
     */
    private void refreshView() {
        invalidate();
    }

    /**
     * 画方格上的文字弹框
     * @param canvas 画布
     */
    private void drawPopupInfo(Canvas canvas) {
        if (clickDay != null) {
            //先根据方格来画出一个小三角形，坐标就是方格的中间
            Path infoPath = new Path();
            //先从方格中心
            infoPath.moveTo(clickDay.startX + boxSide / 2, clickDay.startY + boxSide / 2);
            //然后是方格的左上点
            infoPath.lineTo(clickDay.startX, clickDay.startY);
            //然后是方格的右上点
            infoPath.lineTo(clickDay.endX, clickDay.startY);
            //画出三角
            canvas.drawPath(infoPath,infoPaint);
            //画三角上的圆角矩形
            textPaint.setColor(Color.WHITE);
            //得到当天的文本信息
            String popupInfo = clickDay.toString();
            System.out.println(popupInfo);
            //计算文本的高度和长度用以确定矩形的大小
            float infoHeight = metrics.descent - metrics.ascent;
            float infoLength = textPaint.measureText(popupInfo);
            Log.e("height",infoHeight+"");
            Log.e("length",infoLength+"");
            //矩形左上点应该是x=当前天的x+边长/2-（文本长度/2+文本和框的间隙）
            float leftX = (clickDay.startX + boxSide / 2 ) - (infoLength / 2 + boxSide);
            //矩形左上点应该是y=当前天的y+边长/2-（文本高度+上下文本和框的间隙）
            float topY = clickDay.startY-(infoHeight+2*boxSide);
            //矩形的右下点应该是x=leftX+文本长度+文字两边和矩形的间距
            float rightX = leftX+infoLength+2*boxSide;
            //矩形的右下点应该是y=当前天的y
            float bottomY = clickDay.startY;
            System.out.println(""+leftX+"/"+topY+"/"+rightX+"/"+bottomY);
            RectF rectF = new RectF(leftX, topY, rightX, bottomY);
            canvas.drawRoundRect(rectF,4,4,infoPaint);
            //绘制文字,x=leftX+文字和矩形间距,y=topY+文字和矩形上面间距+文字顶到基线高度
            canvas.drawText(popupInfo,leftX+boxSide,topY+boxSide+Math.abs(metrics.ascent),textPaint);
            clickDay = null;//重新置空，保证点击方格外信息消失
            textPaint.setColor(Color.GRAY);//恢复画笔颜色
        }
    }

    /**
     * 设置某天的次数
     * @param year 年
     * @param month 月
     * @param day 日
     * @param contribution 次数
     */
    public void setData(int year,int month,int day,int contribution){
        //先找到是第几天，为了方便不做参数检测了
        for (Day d : mDays) {
            if (d.year == year && d.month == month && d.date == day){
                d.contribution = contribution;
                d.colour = getColour(contribution);
                break;
            }
        }
        refreshView();
    }

    /**
     * 根据提交次数来获取颜色值
     * @param contribution 提交的次数
     * @return 颜色值
     */
    private int getColour(int contribution){
        int colour = 0;
        if (contribution <= 0){
            colour = COLOUR_LEVEL[4];
        }
        if (contribution == 1){
            colour = COLOUR_LEVEL[3];
        }
        if (contribution == 2){
            colour = COLOUR_LEVEL[2];
        }
        if (contribution == 3){
            colour = COLOUR_LEVEL[1];
        }
        if (contribution >= 4){
            colour = COLOUR_LEVEL[0];
        }
        return colour;
    }

}


```

这样弄个布局测试下:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    >

    <com.franky.custom.view.GitHubContributionView
        android:id="@+id/cc_chart"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />
</LinearLayout>


```

随机弄些数据：

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        GitHubContributionView github = (GitHubContributionView) findViewById(R.id.cc_chart);
        github.setData(2016,12,9,2);
        github.setData(2016,11,9,1);
        github.setData(2016,10,5,10);
        github.setData(2016,8,9,3);
        github.setData(2016,4,20,2);
        github.setData(2016,12,13,3);
        github.setData(2016,12,14,3);
        github.setData(2016,2,15,4);
    }
}


```

### 效果

gif 没有录好，看看图片效果：

![](http://upload-images.jianshu.io/upload_images/1627943-7c466df91b10772e.png) 效果. png

### [查看源码](https://link.jianshu.com?t=https://github.com/912419804/CatView/blob/master/app/src/main/java/com/franky/custom/view/GitHubContributionView.java)