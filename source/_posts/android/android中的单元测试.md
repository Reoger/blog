---
title: android中的测试
date: 2018-08-26 16:13:55
categories: android
tags: android,junit
---

# 单元测试
在日常的开发过程中，为了保证代码的质量和减少代码出错的机率，我们常常会通过单元测试来验证我们的代码。
本篇博客就是用来记录单元测试的用法，以免遗忘。

# junit 工具类
在android中编写单元测试的时候，用的最多的还是``junit 4``吧，在新版的android studio中，会默认添加依赖，如果是老项目，可能需要module在``build.gradle``中添加如下的依赖。
```
 testImplementation 'junit:junit:4.12'
```
好了，环境准备完毕，然后就可以开始进行单元测试了。首先我们还是编写一个需要测试的类，简单起见就编写一个``a+b``的实现吧，代码如下：
```
public class Utils {

    public int add(Integer a, Integer b) throws ParseException{
        if (a == null || b== null){
            throw new  ParseException("param is null",0);
        }
        return a + b;
    }
}
```
然后，就可以编写单元测试了，在window环境下，直接按``ctrl+shift+T``即可快速创建一个单元测试。如图：
![图1](http://ovec6nnof.bkt.clouddn.com/2018-08-27_200102.png)

其中的注解我们稍后介绍，
```
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class UtilsTest {

    @Test
    public void testAdd() throws Exception {
        Utils test = new Utils();
        assertEquals(3,test.add(1,3));
    }
}
```
例如我们可以编写一个这样的测试类，使用``@Test``标识这是一个测试，利用``assertEquals``来进行判断结果是否符合预期。
![图2](http://ovec6nnof.bkt.clouddn.com/2018-08-27_200631.png)
很明显，这个测试并没有通过，因为1+3的结果明显不是3，但这是我们的预期的结果就错了，所以就需要更改预期为4，运行后发现测试通过。
下面简单介绍其他的注解和方法的使用。

## 注解

|注解名|注解含义|
|-----|--------|
|@Test|表示此方法为测试方法|
|@Before|在每个测试方法前执行，可做初始化操作|
|@After|在每个测试方法后执行，可做释放资源操作|
|@Ignore|忽略的测试方法|
|@BeforeClass|在类中所有方法前运行。此注解修饰的方法必须是static void|
|@AfterClass|在类中最后运行。此注解修饰的方法必须是static void|
|@RunWith|指定该测试类使用某个运行器|
|@Parameters|指定测试类的测试数据集合|
|@Rule|重新制定测试类中方法的行为|
|@FixMethodOrder|指定测试类中方法的执行顺序|

在测试流程中，上述注解的方法执行的顺序为：**@BeforeClass –> @Before –> @Test –> @After –> @AfterClass**
主语其他的非流程类的测试注解，**@RunWith、@Parameters、@Rule、@FixMethodOrder**，稍后会进一步介绍其用法。

### 使用``@Parameters``配置多组测试数据
在需要进行多组数据进行测试的时候，我们可能会这么写：
```
 assertArrayEquals(new int[]{3,5,9},new int[]{test.add(1,2),test.add(2,3),test.add(4,5)});
```
是的，上述方式可以实现测试多组数据，我们甚至可以将他们分开一条一条的测试，以便于我们快速找到测试不通过的地方。但是有一种更优雅的方式，我们可以这么写：
```
@RunWith(Parameterized.class)
public class UtilsTest {
    public Point point;

    public UtilsTest(Point point) {
       this.point = point;
    }

    @Test
    public void testAdd() throws Exception {
        Utils test = new Utils();
        assertEquals(point.x+point.y,test.add(point.x,point.y));
    }

    @Parameterized.Parameters
    public static Collection params(){
        return Arrays.asList(new Point(1,2),new Point(3,4),new Point(5,6));
    }
}
```
可以看到，我们在``UtilsTest``，添加了注解``@RunWith(Parameterized.class)``，用于表示次测试类的参数是可变的，然后添加了一个有参的构造方法`` public UtilsTest(Point point) ``，用于接受可变的参数，最后使用``@Parameterized.Parameters`` 来构造参数列表，如此我们的测试类就可以实现一次运行多个测试样例了。结果如下所示：
![图3](http://ovec6nnof.bkt.clouddn.com/2018-08-27_205446.png)

### 测试抛出异常
当我们需要测试传入异常参数时，是否会如预想的一样抛出异常，这个时候我们就可以使用到``@test``注解中的``expected``属性，例如，我们还是测试上面的a+b,测试代码如下：
```
    @Test(expected = ParseException.class)
    public void testAdd() throws Exception {
        Utils test = new Utils();
        assertEquals(5,test.add(null,5));
    }
```
在测试类中，我们传入了``null``到``add``方法中，我们预想这种肯定会出现异常，所以我们在``Test``注解中添加了``expected``来捕获``ParseException``异常，如果捕获到这个异常，此时通过，如果没有捕获到这个异常，则这个测试方法会主动抛出``ParseException``异常，并说名测试不通过。


### Assert类的主要用法

|方法名	|方法描述|
------|--------
assertEquals	|断言传入的预期值与实际值是相等的
assertNotEquals|	断言传入的预期值与实际值是不相等的
assertArrayEquals|	断言传入的预期数组与实际数组是相等的
assertNull|	断言传入的对象是为空
assertNotNull	|断言传入的对象是不为空
assertTrue|	断言条件为真
assertFalse|	断言条件为假
assertSame	|断言两个对象引用同一个对象，相当于“==”
assertNotSame|	断言两个对象引用不同的对象，相当于“!=”
assertThat|	断言实际值是否满足指定的条件


## 匹配器

|匹配器	|说明	|例子|
|------|-------|----|
|is|	断言参数等于后面给出的匹配表达式|	assertThat(5, is (5));|
|not	|断言参数不等于后面给出的匹配表达式	|assertThat(5, not(6));|
|equalTo|	断言参数相等|	assertThat(30, equalTo(30));|
|equalToIgnoringCase|	断言字符串相等忽略大小写|	assertThat(“Ab”, equalToIgnoringCase(“ab”));|
|containsString|	断言字符串包含某字符串|	assertThat(“abc”, containsString(“bc”));|
|startsWith|	断言字符串以某字符串开始|	assertThat(“abc”, startsWith(“a”));|
|endsWith	|断言字符串以某字符串结束	|assertThat(“abc”, endsWith(“c”));|
|nullValue|	断言参数的值为null|	assertThat(null, nullValue());|
|notNullValue	|断言参数的值不为null|	assertThat(“abc”, notNullValue());|
|greaterThan|	断言参数大于	|assertThat(4, greaterThan(3));|
|lessThan|	断言参数小于|	assertThat(4, lessThan(6));|
|greaterThanOrEqualTo	|断言参数大于等于|	assertThat(4, greaterThanOrEqualTo(3));|
|lessThanOrEqualTo|	断言参数小于等于|	assertThat(4, lessThanOrEqualTo(6));|
|closeTo|	断言浮点型数在某一范围内|	assertThat(4.0, closeTo(2.6, 4.3));|
|allOf|	断言符合所有条件，相当于&&|	assertThat(4,allOf(greaterThan(3), lessThan(6)));|
|anyOf|	断言符合某一条件，相当于或|	assertThat(4,anyOf(greaterThan(9), lessThan(6)));|
|hasKey|	断言Map集合含有此键|	assertThat(map, hasKey(“key”));|
|hasValue|	断言Map集合含有此值|	assertThat(map, hasValue(value));|
|hasItem	|断言迭代对象含有此元素|	assertThat(list, hasItem(element));|


# UI测试
上述的单元测试虽然功能很强大，但是局限性也很大，比如不能测试UI相关的，不能测试与``Context``相关的方法。我们可以利用Ui测试来进行这方面的测试。
关于ui测试，请参考这篇[博客](https://juejin.im/post/5b66de2c6fb9a04fbd1b4725)，实在写的太好了。
关于android studio中的Ui测试，可以参考[官方文档](https://developer.android.com/studio/test/espresso-test-recorder)。
这里仅记录一下使用心得：
## espresso必要的依赖
在``module``中的``build.gradle``中添加下面三条依赖：
```
androidTestImplementation 'com.android.support.test:runner:1.0.2'
androidTestImplementation 'com.android.support.test:rules:1.0.2'
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
```
使用示例：
譬如在``MainActivity``中添加有一个如下的方法：
```
public class MainActivity extends AppCompatActivity  {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    /**
    * 需要测试的方法,简单的调起浏览器打开baidu首页
    **/
    public static void startActiviy(Context context){
        Uri uri = Uri.parse("http://wwww.baidu.com");
        Intent t = new Intent(Intent.ACTION_VIEW);
        t.setData(uri);
        t.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(t);
    }
}
```
然后，我们还是可以选择``ctrl+shift+T``来创建测试类，选择``android test``目录即可。
然后我们可以编写一个类似于这样类来进行测试：
```
/**
 * Instrumented test, which will execute on an Android device.
 *
 * @see <a href="http://d.android.com/tools/testing">Testing documentation</a>
 */
@RunWith(AndroidJUnit4.class)
public class ExampleInstrumentedTest {
    @Test
    public void useAppContext() throws Exception {
        // Context of the app under test.
        Context appContext = InstrumentationRegistry.getTargetContext();

        assertEquals("com.example.cm.testpulgin", appContext.getPackageName());
    }

    @Test
    public void testOpenUrl(){
        MainActivity.startActiviy(InstrumentationRegistry.getTargetContext());
    }

    @Test
    public void test1(){
        assertEquals(1,1);
    }
}
```
我们选择运行，选择目标手机，即可在手机上实现测试。这里关键的一点是利用``InstrumentationRegistry.getTargetContext()``获取到了``Context``对象。
我们可以利用这个``Context``对象来测试与手机密切相关的一些属性了，前面的单元测试只能测试代码在jvm上是否运行正常，而到了这里我们就可以测试代码是否在手机上运行正常了。
譬如我们要测试一些用户的操作的行为是否符合预期，我们可以写一个类似于这样的测试类：
```

@LargeTest
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    @Rule
    public ActivityTestRule<MainActivity> mActivityTestRule = new ActivityTestRule<>(MainActivity.class);

    @Test
    public void mainActivityTest() {
        ViewInteraction button = onView(
                allOf(withId(R.id.but1),
                        childAtPosition(
                                childAtPosition(
                                        withId(android.R.id.content),
                                        0),
                                0),
                        isDisplayed()));
        button.check(matches(isDisplayed()));

        ViewInteraction appCompatButton = onView(
                allOf(withId(R.id.but1), withText("测试hook-ams"),
                        childAtPosition(
                                childAtPosition(
                                        withId(android.R.id.content),
                                        0),
                                0),
                        isDisplayed()));
        appCompatButton.perform(click());

    }

    private static Matcher<View> childAtPosition(
            final Matcher<View> parentMatcher, final int position) {

        return new TypeSafeMatcher<View>() {
            @Override
            public void describeTo(Description description) {
                description.appendText("Child at position " + position + " in parent ");
                parentMatcher.describeTo(description);
            }

            @Override
            public boolean matchesSafely(View view) {
                ViewParent parent = view.getParent();
                return parent instanceof ViewGroup && parentMatcher.matches(parent)
                        && view.equals(((ViewGroup) parent).getChildAt(position));
            }
        };
    }
}
```

对其中的一些方法进行简单说明：

|方法名|含义|
|-----|----|
|click() |点击view|
|clearText()|清除文本内容|
|swipeLeft()|从右往左滑|
|swipeRight()|从左往右滑|
|swipeDown()|从上往下滑|
|swipeUp()|从下往上滑|
|click()|点击view|
|closeSoftKeyboard()|关闭软键盘|
|pressBack()|按下物理返回键|
|doubleClick()|双击|
|longClick()|长按|
|scrollTo()|滚动|
|replaceText()|替换文本|
|openLinkWithText()|打开指定超链|

# 参考链接
* [Android单元测试(一)：JUnit框架的使用](https://blog.csdn.net/qq_17766199/article/details/78243176)
* [使用 Espresso 测试记录器创建界面测试](https://developer.android.com/studio/test/espresso-test-recorder)
* [测试应用](https://developer.android.com/studio/test/#run_a_test)
* [解放双手 - Android 开发应该尝试的 UI 自动化测试](https://juejin.im/post/5b66de2c6fb9a04fbd1b4725)



