# Android-Note
一些Android学习的笔记，知识点，欢迎拍砖，指出错误

#目录
* [1.广播的安全性问题以及本地广播](#广播的安全性问题以及本地广播)
* [2.主线程向子线程发送消息](#主线程向子线程发送消息)
* [3.Android空格对齐问题](#Android空格对齐问题)









### 广播的安全性问题以及本地广播

  * 大家都知道广播是系统级别的可以在应用间传播，应用A发送的广播应用B也可以接收到，但有时候我们传递的数据不希望别人接收到哪怎么办呢？
    官网有段原话:  
  'When you use sendBroadcast(Intent) or related methods, normally any other application can receive these broadcasts. You can control who can receive such broadcasts through permissions described below. Alternatively, starting with ICE_CREAM_SANDWICH, you can also safely restrict the broadcast to a single application with Intent.setPackage'    
  意思就是在4.0之后可以在Intent上设置Intent.setPackage，意思是可以指定接收该广播的应用包名。设置后只有对应该包名的应用才可以接收到这个广播，如果设置的是自己的包名，那么就只有自己能收到该广播了。
  * 第二种方法是使用v4包里面的本地广播LocalBroadcastManager。因为本地广播LocalBroadcastManager实质是Handler加接口会调的方式，数据在本应用   范围内传播，不用担心隐私数据泄露的问题。 不用担心别的应用伪造广播，造成安全隐患。 相比在系统内发送全局广播，它更高效。  

### 主线程向子线程发送消息
--
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
 3. 你发现这个Handler属于子线程但是handleMessage里面是可以更新UI的，这一点不是很理解，请知道的同学能不吝赐教。
 
### Android空格对齐问题
--
Android中布局经常有这样的布局形式：

直接输入两个空格会合并成一个空格，如果是4个字和两个字就没法对齐了


|#|名称|编号|描述|
|---|----|-----|------
|1| ```&nbsp;``` | ```&#160;```   | 不断行的空格，长度与常规空格相同
|2| ```&ensp;``` | ```&#8194;```  | 半角空格，长度等于半个中文字符
|3| ```&emsp;``` | ```&#8195;```  | 全角空格，长度等于一个中文字符
