# Android-Note
一些Android学习的笔记，知识点，欢迎拍砖，指出错误

#目录
* [1.广播的安全性问题以及本地广播](#广播的安全性问题以及本地广播)
* [2.主线程向子线程发送消息](#主线程向子线程发送消息)
* [3.空格对齐问题](#空格对齐问题)
* [4.使用fillViewport解决布局不能撑满全屏的问题](#使用fillViewport解决布局不能撑满全屏的问题)
* [5.clipToPadding和clipChildren的作用](#clipToPadding和clipChildren的作用)

### 广播的安全性问题以及本地广播

  * 大家都知道广播是系统级别的可以在应用间传播，应用A发送的广播应用B也可以接收到，但有时候我们传递的数据不希望别人接收到哪怎么办呢？
    官网有段原话:  
  'When you use sendBroadcast(Intent) or related methods, normally any other application can receive these broadcasts. You can control who can receive such broadcasts through permissions described below. Alternatively, starting with ICE_CREAM_SANDWICH, you can also safely restrict the broadcast to a single application with Intent.setPackage'    
  意思就是在4.0之后可以在Intent上设置Intent.setPackage，意思是可以指定接收该广播的应用包名。设置后只有对应该包名的应用才可以接收到这个广播，如果设置的是自己的包名，那么就只有自己能收到该广播了。
  * 第二种方法是使用v4包里面的本地广播LocalBroadcastManager。因为本地广播LocalBroadcastManager实质是Handler加接口会调的方式，数据在本应用   范围内传播，不用担心隐私数据泄露的问题。 不用担心别的应用伪造广播，造成安全隐患。 相比在系统内发送全局广播，它更高效。  

### 主线程向子线程发送消息

一般我们Handler是用在子线程执行耗时任务后发送消息到主线程更新UI的，但是能不能主线程向子线程发送消息呢？答案是可以的。  
 1. 首先主线程向子线程发送消息的话，就必须要让Handler属于子线程，就要在子线程创建这个Handler.直接new Handler会报错java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()，提示缺少Looper。
  ```java
  new Thread( ){
              @Override
              public void run() {
                 handler = new Handler(){
                      @Override
                      public void handleMessage(Message msg) {
                          super.handleMessage(msg);
                      }
                  };
  
              }
          }.start();
  ```  
   主线程为什么能直接new，因为主线程创建的时候系统为我们绑定了Looper对象，网上Handler机制文章很多写的也比较详细。
 2. 那就手动给他准备个Looper
 ```java
  new Thread( ){
            @Override
            public void run() {
                Looper.prepare();

               handler = new Handler(){
                    @Override
                    public void handleMessage(Message msg) {
                        super.handleMessage(msg);

                        //do something
                        Toast.makeText(MainActivity.this, "handleMessage", Toast.LENGTH_SHORT).show();

                    }
                };

                Looper.loop();
            }
        }.start();
```
 这样就可以了，还要注意的是 Looper.prepare()与 Looper.loop()是一个回路，Looper.loop()之后的代码是执行不到的。  
 3. 你发现这个Handler属于子线程，可以在handleMessage里面弹Toast，但是不可以更新UI的。

### 空格对齐问题

Android中布局经常有这样的布局形式：

![](https://github.com/nanixiaoseng/Android-Note/blob/master/img/img_login_dq.png "img")

直接输入两个空格会合并成一个空格，如果是4个字和两个字就没法对齐了


|#|名称|编号|描述|
|---|----|-----|------
|1| ```&nbsp;``` | ```&#160;```   | 不断行的空格，长度与常规空格相同
|2| ```&ensp;``` | ```&#8194;```  | 半角空格，长度等于半个中文字符
|3| ```&emsp;``` | ```&#8195;```  | 全角空格，长度等于一个中文字符
注：使用编号就可以了  

### 使用fillViewport解决布局不能撑满全屏的问题

项目中经常有这样的场景：中间内容长度不固定，当内容不够一屏幕底下两个按钮要在屏幕底部，当内容超过一屏幕按钮在内容下面，滑动后才能显示按钮。

![](https://github.com/nanixiaoseng/Android-Note/blob/master/img/scrollview_fillveewport1.png) 

![](https://github.com/nanixiaoseng/Android-Note/blob/master/img/scrollview_fillveewport2.png)

下面是一个简单的实现方法：
```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fillViewport="true">
    <LinearLayout android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="8dp"
            android:text="用户协议"/>

        <View android:layout_width="match_parent"
            android:layout_height="1dp"
            android:layout_marginEnd="@dimen/activity_horizontal_margin"
            android:layout_marginLeft="@dimen/activity_horizontal_margin"
            android:layout_marginRight="@dimen/activity_horizontal_margin"
            android:layout_marginStart="@dimen/activity_horizontal_margin"
            android:background="@color/colorPrimary"
            android:paddingTop="@dimen/activity_vertical_margin"/>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:lineSpacingExtra="5dp"
            android:text="@string/agreement"
            android:textColor="@color/colorPrimary"/>
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <View android:id="@+id/view_mid"
                android:layout_width="0dp"
                android:layout_height="1dp"
                android:layout_centerHorizontal="true"/>

            <Button android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_toLeftOf="@id/view_mid"
                android:layout_toStartOf="@id/view_mid"
                android:text="同意"/>

            <Button android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_toEndOf="@id/view_mid"
                android:layout_toRightOf="@id/view_mid"
                android:text="取消"/>
        </RelativeLayout>
    </LinearLayout>
</ScrollView>
```
把```android:fillViewport="true"```改为false大家可以试一下效果是相差很大的。

### clipToPadding和clipChildren的作用

clipToPadding:控件的绘制区域是否在padding里面, 值为true时padding那么绘制的区域就不包括padding区域;

clipChildren:当ViewGroup的Padding不为0时，定义ViewGroup是否裁剪子孩子的填充。

这两个属性默认是true的，所以在设置了padding情况下，视图默认绘制是在padding内部的，把这两个属性设置了false那么这样子控件就能画到padding的区域了。
