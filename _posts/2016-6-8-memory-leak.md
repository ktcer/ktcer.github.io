---
layout: post
title: android开发中，可能会导致内存泄露的问题
categories: Android
description: android内存泄露的问题
keywords: Android
---

#android开发中，可能会导致内存泄露的问题


 

在android编码中，会有一些简便的写法和编码习惯，会导致我们的代码有很多内存泄露的问题。
在这里做一个已知错误的总结（其中有一些是个人总结和参考其他博主的文章，在此表示感谢）。

----------

本文会不定时更新，将自己遇到的内存泄漏相关的问题记录下来并提供解决办法。

内存泄露的原因：

资源对象没关闭。
如Cursor、File等资源。他们会在finalize中关闭，但这样效率太低。容易造成内存泄露。
SQLiteCursor，当数据量大的时候容易泄露
使用Adapter时，没有使用系统缓存的converView。
即时调用recycle（）释放不再使用的Bitmap。
适当降低Bitmap的采样率，如：
```
BitmapFactory.Options options = newBitmapFactory.Options();    
options.inSampleSize = 2;//图片宽高都为原来的二分之一，即图片为原来的四分之一    
Bitmap bitmap =BitmapFactory.decodeStream(cr.openInputStream(uri), null, options); preview.setImageBitmap(bitmap);

```

使用application的context来替代activity相关的context。

这是两种不同的context，也是最常见的两种.第一种中context的生命周期与Application的生命周期相关的，context随着Application的销毁而销毁，伴随application的一生，与activity的生命周期无关.第二种中的context跟Activity的生命周期是相关的，但是对一个Application来说，Activity可以销毁几次，那么属于Activity的context就会销毁多次.至于用哪种context，得看应用场景，个人感觉用Activity的context好一点，不过也有的时候必须使用Application的context.application context可以通过
Context.getApplicationContext或者Activity.getApplication方法获取.
还有就是，在使用context的时候，小心内存泄露，防止内存泄露，注意一下几个方面：
　1. 不要让生命周期长的对象引用activity context，即保证引用activity的对象要与activity本身生命周期是一样的
　2. 对于生命周期长的对象，可以使用application context
　3. 避免非静态的内部类，尽量使用静态类，避免生命周期问题，注意内部类对外部对象引用导致的生命周期变化
尽量避免activity的context在自己的范围外被使用，这样会导致activity无法释放。
注册没取消造成内存泄露
如：广播
集合中的对象没清理造成的内存泄露我们通常把一些对象的引用加入到了集合中，当我们不需要该对象时，并没有把它的引用从集合中清理掉，这样这个集合就会越来越大。如果这个集合是static的话，那情况就更严重了。
Handler应该申明为静态对象， 并在其内部类中保存一个对外部类的弱引用。如下：

```
static class MyHandler extends Handler 
{
       WeakReference mActivityReference;
       MyHandler(Activity activity)
      { 
            mActivityReference= new WeakReference(activity);
      }
     @Override
     public void handleMessage(Message msg)
    {
           final Activity activity = mActivityReference.get();
           if (activity != null)
          {
                 mImageView.setImageBitmap(mBitmap);
          }    
     }
}
```

1，编写单例的时候常出现的错误。

     错误方式：

```
     public class Foo{
          private static Foo foo;
          private Context mContext;
          private Foo(Context mContext){
               this.mContext = mContext;
          }

          // 普通单例，非线程安全
          public static Foo getInstance(Context mContext){
               if(foo == null)
                    foo = new Foo(mContext);
               return foo;
          }

          public void otherAction(){
               mContext.xxxx();
 }
              
```

 

          


     错误原因：

     如果我们在Activity A中或者其他地方使用Foo.getInstance()时，我们总是会顺手写一个『this』或者『mContext』（这个变量也是指向this）。试想一下，当前我们所用的Foo是单例，意味着被初始化后会一直存在与内存中，以方便我们以后调用的时候不会在此次创建Foo对象。但Foo中的『mContext』变量一直都会持有Activity A中的『Context』，导致Activity A即使执行了onDestroy方法，也不能够将自己销毁。但『applicationContext』就不同了，它一直伴随着我们应用存在（中途也可能会被销毁，但也会自动reCreate），所以就不用担心Foo中的『mContext』会持有某Activity的引用，让其无法销毁。



```
     //正确方式：

     public class Foo{
          private static Foo foo;
          private Context mContext;
          private Foo(Context mContext){
               this.mContext = mContext;
          }
          // 普通单例，非线程安全
          public static Foo getInstance(Context mContext){
               if(foo == null)
                    foo = new Foo(mContext.getApplicationContext());
               return foo;
          }

          public void otherAction(){
               mContext.xxxx();
               ….

          }

     }
```
     

2，使用匿名内部类的时候经常出现的错误

```
     错误方式：

     public class FooActivity extends Activity{
          private TextView textView;          
         private Handler handler = new Handler(){
               @override
               public void handlerMessage(Message msg){

               }
          };

          @override

          public void onCreate(Bundle bundle){
               super.onCreate(bundle);
               setContextView(R.layout.activity_foo_layout);
          
               textView = (TextView)findViewById(R.id.textView);
               handler.postDelayed(new Runnable(){
                    @override
                    public void run(){
                         textView.setText(“ok”);
                    };
               },1000 * 60 * 10);
          }
     }
```



     错误原因：

     当我们执行了FooActivity的finish方法，被延迟的消息会在被处理之前存在于主线程消息队列中10分钟，而这个消息中又包含了Handler的引用，而Handler是一个匿名内部类的实例，其持有外面的FooActivity的引用，所以这导致了FooActivity无法回收，进而导致FooActivity持有的很多资源都无法回收，所以产生了内存泄露。

     注意上面的new Runnable这里也是匿名内部类实现的，同样也会持有FooActivity的引用，也会阻止FooActivity被回收。

     一个静态的匿名内部类实例不会持有外部类的引用。
```
     //正确方式：

     public class FooActivity extends Activity{
          private TextView textView;
          private static class MyHandler extends Handler {
          private final WeakReference<FooActivity> mActivity;
          public MyHandler(FooActivity activity) {
               mActivity = new WeakReference<FooActivity>(activity);
          }
          
          @Override
          public void handleMessage(Message msg) {
               FooActivity activity = mActivity.get();
                    if (activity != null) {
                         // ...
                    }
               }
          }

          private final MyHandler handler = new MyHandler(this);
            @override
          public void onCreate(Bundle bundle){
               super.onCreate(bundle);
               setContextView(R.layout.activity_foo_layout);
                textView = (TextView)findViewById(R.id.textView);
               handler.postDelayed(new MyRunnable(textView),1000 * 60 * 10);
          }

          private static class MyRunnable implements Runnable{
               private WeakReference<TextView> textViewWeakReference;
              public MyRunnable(TextView textView){
                    textViewWeakReference = new WeakReference<TextView>(textView);

               }

                    @override
                    public void run(){
                         final TextView textView = textViewWeakReference.get();
                         if(textView != null){
                              textView.setText("OK");
                         }
                    };
          }
     }
```
     

3，在使用handler后，记得在onDestroy里面handler.removeCallbacksAndMessages(object token);

     handler.removeCallbacksAndMessages(null);

     // removeCallbacksAndMessages,当参数为null的时候，可以清除掉所有跟次handler相关的Runnable和Message，我们在onDestroy中调用次方法也就不会发生内存泄漏了。




开发中需要注意的点以免内存泄漏：

     1，不要让生命周期长于Activity的对象持有到Activity的引用

     2，尽量使用Application的Context而不是Activity的Context

     3，尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用（具体可以查看细话Java：”失效”的private修饰符了解）。如果使用静态内部类，将外部实例引用作为弱引用持有。

     4，垃圾回收不能解决内存泄露，了解Android中垃圾回收机制




获取context的方法，以及使用上context和applicationContext的区别：

     1，View.getContext,返回当前View对象的Context对象，通常是当前正在展示的Activity对象。

     2，Activity.getApplicationContext,获取当前Activity所在的(应用)进程的Context对象，通常我们使用Context对象时，要优先考虑这个全局的进程Context。

     3，ContextWrapper.getBaseContext():用来获取一个ContextWrapper进行装饰之前的Context，可以使用这个方法，这个方法在实际开发中使用并不多，也不建议使用。

     4，Activity.this 返回当前的Activity实例，如果是UI控件需要使用Activity作为Context对象，但是默认的Toast实际上使用ApplicationContext也可以。

![使用](http://img.blog.csdn.net/20160608110058147)

     大家注意看到有一些NO上添加了一些数字，其实这些从能力上来说是YES，但是为什么说是NO呢？下面一个一个解释：

     数字1：启动Activity在这些类中是可以的，但是需要创建一个新的task。一般情况不推荐。

     数字2：在这些类中去layout inflate是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。

     数字3：在receiver为null时允许，在4.2或以上的版本中，用于获取黏性广播的当前值。（可以无视）

     注：ContentProvider、BroadcastReceiver之所以在上述表格中，是因为在其内部方法中都有一个context用于使用。

     好了，这里我们看下表格，重点看Activity和Application，可以看到，和UI相关的方法基本都不建议或者不可使用Application，并且，前三个操作基本不可能在Application中出现。实际上，只要把握住一点，凡是跟UI相关的，都应该使用Activity做为Context来处理；其他的一些操作，Service,Activity,Application等实例都可以，当然了，注意Context引用的持有，防止内存泄漏。

