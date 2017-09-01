---
layout: post
title: "Android单元测试和UI测试"
description: "Android单元测试和UI测试"
category: Android
tags: [test]
---

原文出处[Unit and UI Testing in Android Studio](https://io2015codelabs.appspot.com/codelabs/android-studio-testing#1)

文章参考：[android-testing-support-library](https://google.github.io/android-testing-support-library/docs/index.html)

本文将教你编写Java的单元测试JUnit 和编写运行在手机上的Espresso测试

## **开始Junit单元测试.**

首先需要在Build Variants的窗口选择Test Artifact，设置为Unit Tests
如图所示：

![p1.png](http://i2.buimg.com/567571/2bc3d33cae1e0e2a.png)

在src下创建test文件夹，新建项目Android stadio会自动为我们生成，项目的目录结构如下。

![p2.png](http://i2.buimg.com/567571/5d6f826c935a28fb.png)

然后添加JUnit4依赖

	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
	    compile 'com.android.support:appcompat-v7:23.4.0'
	}

开始JUnit测试
先编写一个简单的被测试的类

	public class Calculator {
	
	    public double sum(double a, double b){
	        return a+b;
	    }
	
	    public double substract(double a, double b){
	        return a-b;
	    }
	
	    public double divide(double a, double b){
	        return a/b;
	    }
	
	    public double multiply(double a, double b){
	        return a*b;
	    }
	}

只需在编辑器内右键点击Calculator类的声明，选择Go to > Test，然后"Create a new test…"

![p3.png](http://i2.buimg.com/567571/6989a2373390d52e.png)

选择Create New Test

![p4.png](http://i2.buimg.com/567571/11a7fa43b390bdae.png)

在窗口中选择Junit4，修改测试类名，选中setUp/@Before，添加需要测试的方法。

![p5.png](http://i2.buimg.com/567571/3200c6c625b4fbec.png)

ok后会自动生成测试类文件。在相应的方法中添加测试用例，结果假设。

	public class CalculatorTest {
	
	    Calculator calculator;
	
	    @Before
	    public void setUp() throws Exception {
	        calculator = new Calculator();
	    }
	
	    @Test
	    public void testSum() throws Exception {
	        assertEquals(6d,calculator.sum(1d,5d),0);
	    }
	
	    @Test
	    public void testSubstract() throws Exception {
	        assertEquals(1d, calculator.substract(5d, 4d), 0);
	    }
	
	    @Test
	    public void testDivide() throws Exception {
	        assertEquals(4d, calculator.divide(20d, 5d), 0);
	    }
	
	    @Test
	    public void testMultiply() throws Exception {
	        assertEquals(10d, calculator.multiply(2d, 5d), 0);
	    }
	}

选择测试类，右键选择run就可以运行JUnit测试，关于方法的执行顺序是每一个有@before标注的都会在@Test标注的方法执行前执行一遍。具体的语法也比较简单。

也可以在命令行输入gradle指令

	./gradlew test
	
 所有假设都正确就可以代表测试通过会显示Process finished with exit code 0，同时在被测试的类会显示测试覆盖率。断言失败将会显示
 
	 java.lang.AssertionError: 
	Expected :43.0
	Actual   :6.0
	   <Click to see difference>

可以查看到哪里测试没有通过。
可能你已经注意到了Android Studio从来没有让你连接设备或者启动模拟器来运行测试。那是因为，位于src/tests目录下的测试是运行在本地电脑Java虚拟机上的单元测试。编写测试，实现功能使测试通过，然后再添加更多的测试...这种工作方式使快速迭代成为可能，我们称之为测试驱动开发。

下面讲到的instrumentation测试

##**Instrumentation测试**

测试库包含Espresso，Espresso是一个提供了简单 API 的用于 android app UI 测试的测试框架。让我们通过编辑build.gradle的相关部分来把它们添加进我们的工程。需要先把Test Artifact 修改为Android Instrumentation Tests。

	apply plugin: 'com.android.application'
	
	android {
	    compileSdkVersion 23
	    buildToolsVersion "24.0.0"
	
	    defaultConfig {
	        applicationId "com.example.zhangtao21.testdemo"
	        minSdkVersion 19
	        targetSdkVersion 23
	        versionCode 1
	        versionName "1.0"
	
	        //ADD THIS LINE:
	        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	
	    //ADD THESE LINES:
	    packagingOptions {
	        exclude 'LICENSE.txt'
	    }
	}
	
	dependencies {
	    compile fileTree(include: ['*.jar'], dir: 'libs')
	    testCompile 'junit:junit:4.12'
	    compile 'com.android.support:appcompat-v7:23.4.0'
	    compile 'com.jakewharton:butterknife:7.0.1'
	    compile 'com.android.support:recyclerview-v7:24.1.1'
	
	
	    androidTestCompile('com.android.support.test:runner:0.2') {
	        exclude module: 'support-annotations'
	    }
	    androidTestCompile('com.android.support.test:rules:0.2') {
	        exclude module: 'support-annotations'
	    }
	    androidTestCompile('com.android.support.test.espresso:espresso-core:2.1') {
	        exclude module: 'support-annotations'
	    }
	
	    androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.0') {
	        exclude group: 'com.android.support', module: 'appcompat'
	        exclude group: 'com.android.support', module: 'support-v4'
	        exclude module: 'recyclerview-v7'
	        exclude module: 'support-annotations'
	    }
	
	    androidTestCompile ('com.android.support.test.espresso:espresso-intents:2.2.2'){
	        exclude module: 'support-annotations'
	    }
	}
	
由于一些依赖版本冲突，和com.android.support:appcompat-v7的冲突，需要不包含·support-annotations。

添加简单的App交互。
![p6.png](http://i2.buimg.com/567571/6106cb62a5322ac0.png)

###***创建并运行Espresso测试***
在androidTest下创建MainActivityInstrumentationTest

	@RunWith(AndroidJUnit4.class)
	public class MainActivityInstrumentationTest {
	    @Rule
	    public ActivityTestRule<MainActivity> mainActivityActivityTestRule = new ActivityTestRule<MainActivity>(MainActivity.class);
	
	    @Test
	    public void sayHello(){
	        onView(withId(R.id.editText)).perform(typeText("zhangtao"),closeSoftKeyboard());
	        onView(withText("Say hello!")).perform(click());
	        onView(withId(R.id.textView)).check(matches(withText("zhangtao")));
	    }
	}

也可以继承ActivityInstrumentationTestCase2<MainActivity>
实现的操作如下：

+ 找到id为R.id.editText的控件输入文字zhangtao，关闭软键盘
+ 找到文字是Say hello！的控件，点击
+ 找到id为R.id.textView的控件，检测文字是否是zhangtao

这样就会在模拟器或者连接的设备上运行测试，你可以在手机屏幕上看到被执行的动作。最后会在Android Studio输出通过和失败的测试结果。

Espresso 由 3 个主要的组件构成。

这些组件是：

ViewMatchers - 在当前的 view 层级中定位一个 view
ViewActions - 跟你的 view 交互
ViewAssertions - 给你的 view 设置断言
更简单的可以用下面的短语来表述它们：

ViewMatchers – “ 找 某些东西“
ViewActions – “ 做 某些事情“
ViewAssertions – “ 检查 某些东西“

![p7.png](https://camo.githubusercontent.com/31ed007bd41bee4646ad1b28388acd32632ad128/68747470733a2f2f616e64726f696472657365617263682e66696c65732e776f726470726573732e636f6d2f323031352f30332f6573706573736f5f6d61696e5f636f6d706f6e656e74732e706e67)

下面我们要考虑几种情况，如果我们遇到了ListView 该怎么测试？

有人会说直接找到文本然后执行点击，但是如果文本不在一个屏幕内呢，需要滑动。。。但是不知道滑动多少啊，espresso为我们提供了onData，用来操作AdapterView。
如果adapterView里面绑定的是只有String模型的话，我们可以直接判断类型，然后判断是否相等，

        onData(allOf(is(instanceOf((String.class))), is("北京"))).perform(click());
        onView(withId(R.id.text2)).check(matches(withText("city:北京")));
  
执行的操作如下，检测list中所有是String类型的，并且数值是北京的，然后点击，然后判断text2的文本是不是city：北京；

如果不是String类型 我们就需要定义自己的matcher了

	public class CustomMatchers {
	
	    public static Matcher<Object> withStudentname(final String name){
	        return new BoundedMatcher<Object,Student>(Student.class) {
	            @Override
	            public void describeTo(Description description) {
	      //           description.appendText("with name" + name);
	            }
	
	            @Override
	            protected boolean matchesSafely(Student item) {
	                if(name.equals(item.name)){
	                    return true;
	                }
	                return false;
	
	            }
	        };
	    }
	}

我们定义的原则是学生名称匹配。 

	onData(allOf(is(instanceOf((Student.class))),withStudentname("zhangtao")))
		 .inAdapterView(withId(R.id.list)).perform(click());
	onView(withId(R.id.text2)).check(matches(withText("zhangtao:1")));
	 
recyclerView 并不适用于onData，提供了独有的测试方式， 

	onView(withId(R.id.rcview)).perform(RecyclerViewActions.actionOnItem(hasDescendant(withText("zhangtao")), click()));
	onView(withId(R.id.st)).check(matches(withText("zhangtao:1")));
	
测试RecyclerView我们需要使用RecyclerViewActions。

最头疼的就是测试一些同步或者异步的延时任务了，在测试延时任务时，我们需要通知测试在什么时间段开始，通知测试我们需要用到自定义IdlingResource，我们先来看一下我们测试的Activity。

	public class TimeActivity extends AppCompatActivity {
	
	    public boolean isShow = false;
	    TextView textView;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_time);
	        try {
	            Thread.sleep(1000);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        textView = (TextView) findViewById(R.id.time);
	        textView.setVisibility(View.VISIBLE);
	        textView.setText("sleep1000");
	
	        isShow = true;
	    }
	
	    public boolean getShow(){
	        return isShow;
	    }
	}

getShow 是我们外抛的方法，当返回true的时候开始执行测试。

	public class TestIdlingResource implements IdlingResource{
	
	    TimeActivity timeActivity ;
	
	    ResourceCallback resourceCallback;
	
	    TestIdlingResource(TimeActivity timeActivity){
	        this.timeActivity = timeActivity;
	    }
	
	    @Override
	    public String getName() {
	        return TestIdlingResource.class.getName();
	    }
	
	    @Override
	    public boolean isIdleNow() {
	        boolean idle = timeActivity.getShow();
	        if (idle && resourceCallback != null) {
	            resourceCallback.onTransitionToIdle();
	        }
	        return idle;
	    }
	
	    @Override
	    public void registerIdleTransitionCallback(ResourceCallback callback) {
	        this.resourceCallback = callback;
	    }
	
	}

espresso 会检测isIdleNow是否返回tre，true的时候开始执行，resourceCallback.onTransitionToIdle();是必须调用的。

	@RunWith(AndroidJUnit4.class)
	public class TimeActivityTest {
	
	    TestIdlingResource  testIdlingResource;
	
	    @Rule
	    public ActivityTestRule<TimeActivity> rule = new ActivityTestRule<TimeActivity>(TimeActivity.class);
	
	    @Before
	    public void register(){
	        testIdlingResource = new TestIdlingResource(rule.getActivity());
	        Espresso.registerIdlingResources(testIdlingResource);
	    }
	
	    @Test
	    public void testTimer(){
	        onView(withId(R.id.time)).check(matches(withText("sleep1000")));
	    }
	
	    @After
	    public void unregisterIntentServiceIdlingResource() {
	        Espresso.unregisterIdlingResources(testIdlingResource);
	    }
	
	}

在测试开始前生成TestIdlingResource，Espresso调用registerIdlingResource表示开始延时操作，
在结束的时候unregisterIdlingResources。

最后分享一个[google测试示例](https://github.com/googlesamples/android-testing/)

