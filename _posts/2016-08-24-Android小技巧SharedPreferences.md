---
layout: post
title: "Android小技巧SharePreferences"
description: "SharedPreferences"
category: Android
tags: [SharedPreferences]
---

SharedPreferences我相信大家都在熟悉不过，但你对这个东西真的了解吗，先说一下我写这篇文章的情境。

有一个需求，每一次测试使用都要清除SharedPreferences里的数据，这样的话，每测试一次就要卸载应用，或者进入到应用管理，清除应用管理，而且这个操作我相信每一个人都做过，只要需求里带有第一次基本上都这样，还有许多其他的情况。这样做实在是有点厌倦了，所以想在测试中，加一个清除SP的选项。过程还是比较简单的。

SP是存储在应用的shared_prefs文件夹中的， 以现在的专业版项目为例，存储路径为 data/data/com.sankuai.moviepro/shared_prefs文件夹下，是以xml的形式存储，内部是键值对的方式存储，下面是我导出的样例文件。

	<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
	<map>
	    <boolean name="first_open_find_guide" value="false" />
	</map>


插一段不相关的东西：就是Linux上的文件权限，大家有经常看到这样的一串字符，标示文件权限

	drwxr-xr-x
	-rwxrwxrwx

等等类似的，具体的含义如下：
第一个字符时d或者-，d标示是文件夹，-标示非，也就表示是文件。

除去第一个字符剩下的 三个一组，可以分成三组，分别表示的是所有者权限、组权限、其他用户权限
每一组中rwx的含义如下，-代表不具有。

| 权限|简写| 对普通文件的作用 |	   对文件夹的作用  |
| --:|---:| -------------:|------------------:|
| 读取|   r| 	   查看文件内容|列出文件夹中的文件(ls)|
| 写入|   w|     修改文件内容|在文件夹中删除、添加或重命名文件(夹)|
| 执行|   x|文件可以作为程序执行|        cd 到文件夹|


我刚开始想到的方法是直接暴力删除shared_prefs这个文件夹，还尝试了删除该文件夹下的所有文件，然后发现仍然能读取到sp的数据，我开始的时候是以为，对于这个文件夹没有删除权限，后来查了一下是有权限的，查看File Explorer文件确实是被删除了。

然后我换了一种思路，由于sp的name是和文件名相同的，我们遍历该文件架下的所有文件，取到文件名，然后作为sp的name，然后对sp进行操作，执行edit().clear().commit()，这样发现清除sp的效果实现了。代码如下。

                File dir = new File(MovieProApplication.getContext().getFilesDir().getParent() + "/shared_prefs/");
                String[] children = dir.list();
                for (String path : children) {
                    MovieProApplication.getContext().getSharedPreferences(path.replace(".xml", ""), Context.MODE_PRIVATE).edit().clear().commit();
                }
                
那么问题来了 为什么我删除了sp文件却仍然能读到数据呢。没办法了到源码中去寻找答案吧，getSharedPreferences的实现是在ContextImpl中。

	@Override
	   public SharedPreferences getSharedPreferences(String name, int mode) {
	       SharedPreferencesImpl sp;
	       synchronized (ContextImpl.class) {
	           if (sSharedPrefs == null) {
	               sSharedPrefs = new ArrayMap<String, ArrayMap<String, SharedPreferencesImpl>>();
	           }
	
	           final String packageName = getPackageName();
	           ArrayMap<String, SharedPreferencesImpl> packagePrefs = sSharedPrefs.get(packageName);
	           if (packagePrefs == null) {
	               packagePrefs = new ArrayMap<String, SharedPreferencesImpl>();
	               sSharedPrefs.put(packageName, packagePrefs);
	           }
	
	           // At least one application in the world actually passes in a null
	           // name.  This happened to work because when we generated the file name
	           // we would stringify it to "null.xml".  Nice.
	           if (mPackageInfo.getApplicationInfo().targetSdkVersion <
	                   Build.VERSION_CODES.KITKAT) {
	               if (name == null) {
	                   name = "null";
	               }
	           }
	
	           sp = packagePrefs.get(name);
	           if (sp == null) {
	               File prefsFile = getSharedPrefsFile(name);
	               sp = new SharedPreferencesImpl(prefsFile, mode);
	               packagePrefs.put(name, sp);
	               return sp;
	           }
	       }
	       if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
	           getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
	           // If somebody else (some other process) changed the prefs
	           // file behind our back, we reload it.  This has been the
	           // historical (if undocumented) behavior.
	           sp.startReloadIfChangedUnexpectedly();
	       }
	       return sp;
	   }

sSharedPrefs会根据应用名区分，注意他是静态的，包名为key 的packagePrefs就是存储我们平时用得到所有的sp的map，map的key值就是sp的Name，回想一下如果我们关闭应用，kill app，然后我们再进入app，这是sp应该为null，看下面这一段代码，getSharedPrefsFile会根据文件名生成name.xml文件，文件的目录就是shared_prefs

            if (sp == null) {
                File prefsFile = getSharedPrefsFile(name);
                sp = new SharedPreferencesImpl(prefsFile, mode);
                packagePrefs.put(name, sp);
                return sp;
            }
            
getSharedPrefsFile方法：

    public File getSharedPrefsFile(String name) {
        return makeFilename(getPreferencesDir(), name + ".xml");
    }
    
getPreferencesDir，sp目录不存在择生成。

    private File getPreferencesDir() {
        synchronized (mSync) {
            if (mPreferencesDir == null) {
                mPreferencesDir = new File(getDataDirFile(), "shared_prefs");
            }
            return mPreferencesDir;
        }
    }

makeFilename 传入的是sp的目录和spname.xml

    private File makeFilename(File base, String name) {
        if (name.indexOf(File.separatorChar) < 0) {
            return new File(base, name);
        }
        throw new IllegalArgumentException(
                "File " + name + " contains a path separator");
    }

生成的就是一个空文件。然后用这个空文件和mode生成了SharedPreferencesImpl，我们平时拿到的SharedPreferences就是这个类的对象，SharedPreferencesImpl中做了什么，是不是又读取了一些其他的文件呢。

    SharedPreferencesImpl(File file, int mode) {
        mFile = file;
        mBackupFile = makeBackupFile(file);
        mMode = mode;
        mLoaded = false;
        mMap = null;
        startLoadFromDisk();
    }
    
makeBackupFile会生成文件添加.bak后缀的文件

    private static File makeBackupFile(File prefsFile) {
        return new File(prefsFile.getPath() + ".bak");
    }
    
然后看startLoadFromDisk();

    private void startLoadFromDisk() {
        synchronized (this) {
            mLoaded = false;
        }
        new Thread("SharedPreferencesImpl-load") {
            public void run() {
                synchronized (SharedPreferencesImpl.this) {
                    loadFromDiskLocked();
                }
            }
        }.start();
    }
    
这里能看到的就是异步线程执行loadFromDiskLocked

	 private void loadFromDiskLocked() {
	        if (mLoaded) {
	            return;
	        }
	        if (mBackupFile.exists()) {
	            mFile.delete();
	            mBackupFile.renameTo(mFile);
	        }
	
	        // Debugging
	        if (mFile.exists() && !mFile.canRead()) {
	            Log.w(TAG, "Attempt to read preferences file " + mFile + " without permission");
	        }
	
	        Map map = null;
	        StructStat stat = null;
	        try {
	            stat = Os.stat(mFile.getPath());
	            if (mFile.canRead()) {
	                BufferedInputStream str = null;
	                try {
	                    str = new BufferedInputStream(
	                            new FileInputStream(mFile), 16*1024);
	                    map = XmlUtils.readMapXml(str);
	                } catch (XmlPullParserException e) {
	                    Log.w(TAG, "getSharedPreferences", e);
	                } catch (FileNotFoundException e) {
	                    Log.w(TAG, "getSharedPreferences", e);
	                } catch (IOException e) {
	                    Log.w(TAG, "getSharedPreferences", e);
	                } finally {
	                    IoUtils.closeQuietly(str);
	                }
	            }
	        } catch (ErrnoException e) {
	        }
	        mLoaded = true;
	        if (map != null) {
	            mMap = map;
	            mStatTimestamp = stat.st_mtime;
	            mStatSize = stat.st_size;
	        } else {
	            mMap = new HashMap<String, Object>();
	        }
	        notifyAll();
	    }

mBackupFile存在就把mFile删除,然后把mBackupFile替换mFile，这里确实没看懂，究竟要做什么，但知道删除之后文件都是空的，下面就是解析xml内容的操作了。这个流程下来，看起来没什么问题啊，我们关掉app 后应该是读取不到sp内容的。但事实还是读取到了，只能猜了，应该是这里出了问题根本没做这个逻辑。

	  sp = packagePrefs.get(name);
	  if (sp == null) 

也就是说我们关闭app后context并没有释放，我们读取到的数值是静态中存储的数值。
为了保证数据context一定被干掉，我们直接关机，这是我们再打开app确实是读取不到sp的内容了。这样也就解释了为什么删掉文件还是能读取到sp内容的现象了。