---
layout: post
title: “ClassLoader入门”
description: “android”
category: android
tags: [android]
---

# ClassLoader简介

# 双亲委派机制

# JVM中的ClassLoader

# Android中的ClassLoader Dalvik/ART

## PathClassLoader 与 DexClassLoader

## BaseClassLoader解析

# 玩转ClassLoader

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

我这里选用的是 Environment.getExternalStorageDirectory().getPath() 的路径下

![s](/img/gradle/save.png)

## 5.使用

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






