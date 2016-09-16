---
layout: post
title: android双缓冲的原理以及应用场景
categories: Android
description: android双缓冲的原理以及应用场景
keywords: buffer,Android
---

# android双缓冲的原理以及应用场景

    
    
闪烁是图形编程的一个常见问题。当进行复杂的绘制操作时会导致呈现的图像闪烁或具有
其他不可接受的外观。双缓冲的使用解决这些问题。双缓冲使用内存缓冲区来解决由多重
绘制操作造成的闪烁问题。当使用双缓冲时，首先在内存缓冲区里完成所有绘制操作，而
不是在屏幕上直接进行绘图。当所有绘制操作完成后，把内存缓冲区完成的图像直接复制
到屏幕。因为在屏幕上只执行一个图形操作，所以消除了由复杂绘制操作造成的图像闪烁
问题。
    在android 中实现双缓冲,可以使用一个后台画布backcanvas,先把所有绘制操作都在
这上面进行。等图画好了，然后在把backcanvas 拷贝到
与屏幕关联的canvas 上去,如下:

```java
package com.cn.bupt.ktc;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Bitmap.Config;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.drawable.BitmapDrawable;
import android.view.KeyEvent;
import android.view.MotionEvent;
import android.view.View;

import com.cn.fit.R;

/**
 * Created by ktcer on 2016/5/26.
 */
public class TestView extends View implements Runnable {
    /* 声明Bitmap对象 */
    Bitmap mBitQQ = null;

    Paint mPaint = null;

    /* 创建一个缓冲区 */
    Bitmap mSCBitmap = null;

    /* 创建Canvas对象 */
    Canvas backCanvas = null;

    public TestView(Context context) {
        super(context);

        /* 装载资源 */
        mBitQQ = ((BitmapDrawable) getResources().getDrawable(R.drawable.qq)).getBitmap();

        /* 创建屏幕大小的缓冲区 */
        mSCBitmap = Bitmap.createBitmap(320, 480, Config.ARGB_8888);

        /* 创建Canvas */
        backCanvas = new Canvas();

        /* 设置将内容绘制在mSCBitmap上 */
        backCanvas.setBitmap(mSCBitmap);

        mPaint = new Paint();

        /* 将mBitQQ绘制到mSCBitmap上 */
        backCanvas.drawBitmap(mBitQQ, 0, 0, mPaint);

        /* 开启线程 */
        new Thread(this).start();
    }

    public void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        /* 将mSCBitmap显示到屏幕上 */
        canvas.drawBitmap(mSCBitmap, 0, 0, mPaint);
    }

    // 触笔事件
    public boolean onTouchEvent(MotionEvent event) {
        return true;
    }

    // 按键按下事件
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        return true;
    }

    // 按键弹起事件
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        return false;
    }


    public boolean onKeyMultiple(int keyCode, int repeatCount, KeyEvent event) {
        return true;
    }


    /**
     * 线程处理
     */
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            //使用postInvalidate可以直接在线程中更新界面
            postInvalidate();
        }
    }
}
```

