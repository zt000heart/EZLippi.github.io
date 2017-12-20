---
layout: post
title: “ClassLoader，类加载机制解析”
description: “android”
category: android
tags: [android]
---

# ClassLoader简介

任何一个 Java 程序都是由若干个 class 文件组成的一个完整的 Java 程序，在程序运行时，需要将 class 文件加载到 JVM 中才可以使用，负责加载这些 class 文件的就是 Java 的类加载（ClassLoader）机制。
ClassLoader 的作用简单来说就是加载 class 文件，提供给程序运行时使用。

# 双亲委派模型

ClassLoader通过传入父ClassLoader构造

当前的 ClassLoader在加载Class的时候 先判断该类是否被加载过，加载过直接返回，如果没有加载交给父ClassLoader处理。
父ClassLoader也先判断是否加载过，加载过直接返回，没加载过交给祖父ClassLoader处理 依次往上。
如果都没有加载过。祖父ClassLoader尝试从其对应的类路径下寻找 class 字节码文件并载入。如果载入成功，则直接返回 Class，载入失败交给父ClassLoader处理。
父ClassLoader尝试从其对应的类路径下寻找 class 字节码文件并载入。如果载入成功，则直接返回 Class，载入失败交给"当前"ClassLoader处理。

# JVM中的ClassLoader

 BootstrapClassLoader  - 负责加载系统类<br/>
 负责来加载$JAVA_HOME/jre/lib下rt.jar，bootstrap classloader会负责加载java的关键类(java.lang包和其他的运行时类)到内存中。Bootstrap classloader是JVM中的实现<br/>
 ExtClassLoader  - 负责加载扩展类<br/>
 加载lib/ext文件夹下的jar包<br/>
 AppClassLoader  - 负责加载应用类<br/>
 用来加载ClassPath目录下的jar包和Class文件。<br/>

        A a = new A();
        System.out.println(a.getClass().getClassLoader());
        System.out.println(a.getClass().getClassLoader().getParent());
        System.out.println(a.getClass().getClassLoader().getParent().getParent());

sun.misc.Launcher$AppClassLoader@18b4aac2   <br/>
sun.misc.Launcher$ExtClassLoader@1540e19d   <br/>
null   <br/>

# Android中的ClassLoader Dalvik/ART

        Log.e("1"," "+MainActivity.class.getClassLoader());
        Log.e("parent1"," "+MainActivity.class.getClassLoader().getParent());
        Log.e("parent2"," "+MainActivity.class.getClassLoader().getParent().getParent());


12-15 05:19:49.501 1870-1870/com.sankuai.moviepro.activitylife E/1:  dalvik.system.PathClassLoader   <br/>
12-15 05:19:49.501 1870-1870/com.sankuai.moviepro.activitylife E/parent1:  java.lang.BootClassLoader@471b430   <br/>
12-15 05:19:49.501 1870-1870/com.sankuai.moviepro.activitylife E/parent2:  null   <br/>

## PathClassLoader和DexClassLoader

 PathClassLoader 和 DexClassLoader 继承自BaseDexClassLoader  <br/>
 PathClassLoader 在应用启动时创建，从 data/app/… 安装目录下加载 apk 文件。  <br/>
 在 Android 中，App 安装到手机后，apk 里面的 class.dex 中的 class 均是通过 PathClassLoader 来加载的。 <br/>
 BootClassLoader 是 PathClassLoader 的父加载器，其在系统启动时创建，在 App 启动时会将该对象传进来  <br/>
 和java虚拟机中不同的是BootClassLoader是ClassLoader内部类,由java代码实现而不是c++实现,是Android平台上所有ClassLoader的最终parent,这个内部类是包内可见,所以我们没法使用。 <br/>

 对比PathClassLoader只能加载已经安装应用的dex或apk文件，DexClassLoader则没有此限制，可以从SD卡上加载包含class.dex的.jar和.apk 文件，这也是插件化和热修复的基础，在不需要安装应用的情况下，完成需要使用的dex的加载

        //PathClassLoader.java
        public class PathClassLoader extends BaseDexClassLoader {

        37    public PathClassLoader(String dexPath, ClassLoader parent) {
        38        super(dexPath, null, null, parent);
        39    }
        40
        41    /**
        42     * Creates a {@code PathClassLoader} that operates on two given
        43     * lists of files and directories. The entries of the first list
        44     * should be one of the following:
        45     *
        46     * <ul>
        47     * <li>JAR/ZIP/APK files, possibly containing a "classes.dex" file as
        48     * well as arbitrary resources.
        49     * <li>Raw ".dex" files (not inside a zip file).
        50     * </ul>
        51     *
        52     * The entries of the second list should be directories containing
        53     * native library files.
        54     *
        55     * @param dexPath the list of jar/apk files containing classes and
        56     * resources, delimited by {@code File.pathSeparator}, which
        57     * defaults to {@code ":"} on Android
        58     * @param libraryPath the list of directories containing native
        59     * libraries, delimited by {@code File.pathSeparator}; may be
        60     * {@code null}
        61     * @param parent the parent class loader
        62     */
        63    public PathClassLoader(String dexPath, String libraryPath,
        64            ClassLoader parent) {
        65        super(dexPath, null, libraryPath, parent);
        66    }
        67}

 dexPath : 包含 dex 的 jar 文件或 apk 文件的路径集，多个以文件分隔符分隔，默认是“：”<br/>
 libraryPath : 包含 C/C++ 库的路径集，多个同样以文件分隔符分隔，可以为空<br/>

          //DexClassLoader.java
         public class DexClassLoader extends BaseDexClassLoader {
         37    /**
         38     * Creates a {@code DexClassLoader} that finds interpreted and native
         39     * code.  Interpreted classes are found in a set of DEX files contained
         40     * in Jar or APK files.
         41     *
         48     * @param optimizedDirectory directory where optimized dex files
         49     *     should be written; must not be {@code null}
         54     */
         55    public DexClassLoader(String dexPath, String optimizedDirectory,
         56            String libraryPath, ClassLoader parent) {
         57        super(dexPath, new File(optimizedDirectory), libraryPath, parent);
         58    }
         59}

          //BaseDexClassLoader.java
         public class BaseDexClassLoader extends ClassLoader {
         30    private final DexPathList pathList;
         31
         45    public BaseDexClassLoader(String dexPath, File optimizedDirectory,
         46            String libraryPath, ClassLoader parent) {
         47        super(parent);
         48        this.pathList = new DexPathList(this, dexPath, libraryPath, optimizedDirectory);
         49    }
         50
         51    @Override
         52    protected Class<?> findClass(String name) throws ClassNotFoundException {
         53        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
         54        Class c = pathList.findClass(name, suppressedExceptions);
         55        if (c == null) {
         56            ClassNotFoundException cnfe = new ClassNotFoundException("Didn't find class \"" + name + "\" on path: " + pathList);
         57            for (Throwable t : suppressedExceptions) {
         58                cnfe.addSuppressed(t);
         59            }
         60            throw cnfe;
         61        }
         62        return c;
         63    }
           }

               //DexPathList.java
               public Class findClass(String name, List<Throwable> suppressed) {
           317        for (Element element : dexElements) {
           318            DexFile dex = element.dexFile;
           319
           320            if (dex != null) {
           321                Class clazz = dex.loadClassBinaryName(name, definingContext, suppressed);
           322                if (clazz != null) {
           323                    return clazz;
           324                }
           325            }
           326        }
           327        if (dexElementsSuppressedExceptions != null) {
           328            suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
           329        }
           330        return null;
           331    }

DexClassLoader与PathClassLoader都是只有构造方法，唯一不同的是DexClassLoader 多一个参数optimized Directory优化路径，缓存优化后的缓存文件<br/>

# DexClassLoader加载Demo

## 1.创建Project 建立Android Module

包含两个文件ISay.java 和 SayHello.java

package com.sankuai.moviepro.mylibrary;

        import android.content.Context;

        /**
         * Created by zhangtao21 on 2017/12/15.
         */

        public interface ISay {
            void say(Context context);
        }

        package com.sankuai.moviepro.mylibrary;

        import android.content.Context;
        import android.widget.Toast;

        /**
         * Created by zhangtao21 on 2017/12/15.
         */

        public class SayHello implements ISay{
            @Override
            public void say(Context context) {
                Toast.makeText(context, "Hello", Toast.LENGTH_SHORT).show();
            }
        }

## 2.导出jar

gradle 配置Task

        task makeJar(type: Copy) {
            delete 'build/libs/mysdk.jar'
            from('build/intermediates/bundles/release/')
            into('build/libs/')
            include('classes.jar')
            rename ('classes.jar', 'mysdk.jar')
        }

        makeJar.dependsOn(build)

运行  ./gradlew makeJar

![s](/img/gradle/jar.png)

## 3.将jar转换成包含 dex 的jar

使用android sdk 目录下build-tools/任意版本/dx 工具

![s](/img/gradle/dexjar.png)

dx --dex --output=mysdk_dex.jar mysdk.jar

output = 生成的jar文件名 需要转换的jar

看到生成的jar包中已经包含了dex文件

![s](/img/gradle/withdex.png)

## 4.导入手机存储空间目录

使用Android device Monitor 向模拟器导入jar包，路径可以自选 只要你能读取的到

我这里选用的是 Environment.getExternalStorageDirectory().getPath() 的路径下： /storage/emulated/0

![s](/img/gradle/save.png)

## 5.使用DexClassLoader

使用ISay 接口 需要现在项目中添加相同的接口类（类名包名需要相同）

![s](/img/gradle/project.png)

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
//        Log.e("path", ""+Environment.getExternalStorageDirectory().getPath());

        findViewById(R.id.btn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                final File jarFile =
                        new File(Environment.getExternalStorageDirectory().getPath() + File.separator + "mysdk_dex.jar");
                if(!jarFile.canRead()){
                    // 如果没有读权限,确定你在 AndroidManifest 中是否声明了读写权限
                    Log.d("tag", jarFile.canRead() + "");
                    return;
                }
                if (!jarFile.exists()) {
                    Log.e("tag", "mysdk_dex.jar not exists");
                    return;
                }

                DexClassLoader dexClassLoader = new DexClassLoader(jarFile.getAbsolutePath(), getExternalCacheDir().getAbsolutePath(), null, getClassLoader());
                // PathClassLoader dexClassLoader = new PathClassLoader(jarFile.getAbsolutePath(), null, getClassLoader());
                try {
                    Class clazz = dexClassLoader.loadClass("com.sankuai.moviepro.mylibrary.SayHello");
                    ISay say = (ISay) clazz.newInstance();

                    Log.e("type", ""+say.getClass().getSimpleName());
                    say.say(MainActivity.this);
                }catch (Exception e){
                    e.printStackTrace();
                }

            }
        });
        final String[] permissions = {Manifest.permission.WRITE_EXTERNAL_STORAGE};
        findViewById(R.id.per).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ActivityCompat.requestPermissions(MainActivity.this, permissions,10);
            }
        });
    }
}






