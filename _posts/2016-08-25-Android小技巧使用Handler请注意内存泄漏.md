---
layout: post
title: "Android小技巧使用Handler请注意内存泄漏"
description: "Handler"
category: Android，Handler
tags: [Handler]
---

handler使用的很频繁，但使用的过程中需要注意，Handler使用不当，很容易造成内存泄露，在我们的代码中会看到很多这样的写法：

	   Handler handler = new Handler(){
	       @Override
	       public void handleMessage(Message msg) {
	           ....
	       }
	   };
	   handler.sendEmptyMessageDelayed(1,1000);

看起来使用很方便，但这样写很明显的内存泄露。handler发送的消息会放在消息队列中，在这个消息被处理之前会持有Handler的引用，这个Handler由于是非静态内部类，这样这个Handler也会持有Activity的引用，这样就造成了destroy的时候，无法正常释放Activity，所以在使用Handler的时候需要注意两点，有延时操作的时候需要在Destroy的时候调用

	Handler.removeCallbacksAndMessages(null);

对于静态内部类的问题，需要使用弱引用保证内存可以正常释放。

	static class MessageHandler extends Handler {
	        final WeakReference<WelcomeActivity> mActivityWeakReference;
	
	        MessageHandler(WelcomeActivity welcomeActivity) {
	            mActivityWeakReference = new WeakReference<>(welcomeActivity);
	        }
	
	        @Override
	        public void handleMessage(Message msg) {
	            if (mActivityWeakReference.get() != null && !CompatibleUtils.isDestroyed(mActivityWeakReference.get())) {
	                WelcomeActivity welcomeActivity = mActivityWeakReference.get();
	                switch (msg.what) {
	                    case MSG_WELCOME:
	                        welcomeActivity.startMovie();
	                       break;
	                }
	            }
	
	        }
	    }