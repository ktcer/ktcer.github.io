---
layout: post
title: Android MUPDF阅读器放大模糊优化
categories: android
description: MUPDF阅读器放大模糊
keywords: android
---

#### Android MUPDF阅读器放大模糊优化


# Android MUPDF阅读器放大模糊优化


---------

## 前言:
> 问题原因------pdf格式的电子书分三种,定义为A,B,C,A是扫描版,B是非扫描版正常版(格式不是很复杂),C非扫描版不正常版(格式很复杂的那种),mupdf是加载pdf格式速度最快的框架,据了接当前一家大的pdf阅读器运用此框架开发的.存在的问题:mupdf在加载A,B类pdf时速度很快,在加载C类的时候遇到瓶颈,本文解决的是在加载后PDF文本放大过程有一段很长时间的模糊问题.



-------------------

## MUPDF绘制原理

 mupdf在绘制的时候,绘制的Bitmap大小是屏幕尺寸的大小,在	
``` java
public void setPage(int page, PointF size) {}
```
中size是native层获取的pdf尺寸大小传到词方法中.
在setPage方法中有两个重要的字段来计算得到手机屏幕大小的bitmap尺寸:
``` java
mSourceScale = Math.min(mParentSize.x/size.x, mParentSize.y/size.y);
mSize = new Point((int)(size.x*mSourceScale), (int)(size.y*mSourceScale));
```
mSourceScale系数是pdf尺寸映射到手机当前屏幕大小的系数,用它可换算出当前显示的尺寸大小值存在mSize中,在绘制页面的时候根据此mSize来绘制当前的bitmap,绘制的方法是
``` java
mDrawEntire = new AsyncTask<Void,Void,Bitmap>() {
	protected Bitmap doInBackground(Void... v) {
		return drawPage(mSize.x, mSize.y, 0, 0, mSize.x, mSize.y);
			}
```
 在pdf放大后根据缩放比例计算出放大系数重新drawPage
``` java
v[0].bm = drawPage(v[0].patchViewSize.x,v[0].patchViewSize.y,
					v[0].patchArea.left, v[0].patchArea.top,
						v[0].patchArea.width(), v[0].patchArea.height());
```
 
mupdf绘制的总是当前屏幕尺寸大小的bitmap,所以在C类的pdf电子书放大时存在很长一段模糊过程.


## 解决方法
一次性渲染好----放大后uptate拖动不会模糊.
###1.绘制你需要大小的bitmap
修改drawPage方法传入的参数:
``` java
mDrawEntire = new AsyncTask<Void,Void,Bitmap>() {
	protected Bitmap doInBackground(Void... v) {
		return drawPage(mSize.x*2, mSize.y*2, 0, 0, mSize.x*2, mSize.y*2);
			}
```
这里就是一次性绘制好需要大小的bitmap.
###2.更新View
监听放大操作,在放大更新View  执行updaePage()  native方法.不要走drawPage().
``` java
mDrawPatch = new AsyncTask<PatchInfo,Void,PatchInfo>() {
	protected PatchInfo doInBackground(PatchInfo... v) {
		if (!v[0].completeRedraw) {
			v[0].bm = drawPage(v[0].patchViewSize.x, v[0].patchViewSize.y,
			v[0].patchArea.left, v[0].patchArea.top,
			v[0].patchArea.width(), v[0].patchArea.height());
			} else {//放大走此方法
				v[0].bm = updatePage(v[0].bmh, v[0].patchViewSize.x, v[0].patchViewSize.y,
				v[0].patchArea.left, v[0].patchArea.top,
				v[0].patchArea.width(), v[0].patchArea.height());
					}

					return v[0];
				}
```


## 结果

> 一次性渲染好后放大就不会存在一段模糊的过程,此方法以牺牲内存来换取体验速度.(ps,原先占的内存是78MB,优化后测试内存大小是112MB).

  



