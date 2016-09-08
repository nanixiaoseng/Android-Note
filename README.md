# Android-Note
一些Android学习的笔记，知识点，欢迎拍砖

#目录

[1.广播的安全性问题，本地广播和系统广播的区别](#广播的安全性问题以及本地广播LocalBroadcastManager)


####广播的安全性问题以及本地广播LocalBroadcastManager
  * 大家都知道广播是系统级别的可以在应用间传播，应用A发送的广播应用B也可以接收到，但有时候我们传递的数据不希望别人接收到哪怎么办呢？
    官网有段原话：
    When you use sendBroadcast(Intent) or related methods, normally any other application can receive these broadcasts. You can control who can receive such broadcasts through permissions described below. Alternatively, starting with ICE_CREAM_SANDWICH, you can also safely restrict the broadcast to a single application with Intent.setPackage
    意思就是在4.0之后可以在Intent上设置Intent.setPackage，意思是可以指定接收该广播的应用包名。设置后只有对应该包名的应用才可以接收到这个广播，如果设置的是自己的包名，那么就只有自己能收到该广播了。
  * 第二种方法是使用v4包里面的本地广播LocalBroadcastManager。因为本地广播LocalBroadcastManager实质是Handler加接口会调的方式，数据在本应用   范围内传播，不用担心隐私数据泄露的问题。 不用担心别的应用伪造广播，造成安全隐患。 相比在系统内发送全局广播，它更高效。
