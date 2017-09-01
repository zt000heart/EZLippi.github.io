---
layout: post
title: "DialogFragment的坑"
description: "DialogFragment"
category: Android
tags: [DialogFragment]
---

DialogFragment相对于Dialog，它的特点是有自己的生命周期，Android 官方推荐使用 DialogFragment 替代传统的 Dialog，但是使用起来却没有Dialog那么方便了，而且有一些问题我们需要特殊注意。
使用 show() 方法显示 DialogFragment时有时会报出一个异常：

	Caused by: java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
	at android.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1354)
	at android.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1372)
	at android.app.BackStackRecord.commitInternal(BackStackRecord.java:745)
	at android.app.BackStackRecord.commit(BackStackRecord.java:721)
	at android.app.DialogFragment.show(DialogFragment.java:247)
	at com.mw.bookmain.dialog.PushOrder.showOrderDialog(PushOrder.java:656)
	
这个问题的复现步骤是如果show是异步任务控制的，如果用户将应用切换到后台，这是就会报这个错误。改的方式就是使用commitAllowingStateLoss。

	  FragmentManager fragmentManager = activity.getFragmentManager();
	    FragmentTransaction transaction = fragmentManager.beginTransaction();
	    transaction.add(this, tag);
	    transaction.commitAllowingStateLoss();
	    transaction.show(this);
	    
但是我们需要在现实之前判断activity没有finish且dialog没有添加，添加了就移除

	if(loadingDialog.isAdded()){   ((Activity)context).getFragmentManager().beginTransaction().remove(loadingDialog).commit();
	}
	
