# android-jni

仅仅把github当成技术博客来用了，不管了，有时间再去看看怎么上传代码的。<br/>
今天花了一下午的时间去研究一下android的NDK是怎么用的。<br/>
1.参考的资料有：http://developer.android.com/ndk/samples/sample_hellojni.html <br/>
这里是android对于jni的官方资料，主要是讲解如何使用jni实现调用c++的，很实用<br/>
2. 学习之前需要确认好android的ndk/sdk都安装好了，没有安装的话是不行的。
下午先被javah指令坑了半个小时，主要是在环境变量path中只是设置了E:\java1.7.0需要改成E:\java1.7.0\bin <br/>
3.先使用Android Studio建立一个blank activity，在activity中写一个简单的JNI方法,比如
<pre>
public native String getStringFromJNI();//可以考虑使用static，申明成类方法
</pre>
4.写完这个native方法之后，需要生成该方法对应的头文件。在android studio的terminal中输入
<pre>
javah -d jni -classpath ./java com.example.sijunding.ndk_demo.MainActivity
//假设文件是这样的NDK_DEMO/app/src/main/java/com.example.sijunding.ndk_demo/MainActivity
//那么上面的指令应该在main之后这一级输入，这样使用-d jni后生成的jni子文件夹，就和java子文件夹同一级
//-classpath指定了环境变量是哪里，我看到youtube中有加上android.jar包的，还有的support包也加上了，我测试的时候不加可以
//注意./java需要加上去，不加的话提示找不到后面的类，其中com.example.sijunding.ndk_demo是包名，MainActivity是类名
//总之这个地方是很坑爹的，没有看到讲的特别清楚的
</pre>
5.此时应该在jni文件夹中生成了一个头文件，这个头文件是不能改动的。形如
<pre>
JNIEXPORT jstring JNICALL Java_com_example_sijunding_ndk_1demo_MainActivity_getStringFromJNI(JNIEnv*,jobject);
</pre>
6.在jni子文件夹中实现native方法。
<pre>
main.c
=====================================================
#include "com_example_sijunding_ndk_mk_MainActivity.h"
...
return (*env)->NewStringUTF(env,"Hello from JNI!");
</pre>
7.添加Android.mk和Application.mk，这里是参考android的标准教程的，我试过不加的话好像也可以的
<pre>
Android.mk
====================================================================
LOCAL_SRC_FILES :=main.c
LOCAL_MODULE    :=hello-jni
====================================================================
Application.mk
====================================================================
APP_ABI  :=all
====================================================================
这里需要注意的是，android 会默认生成一个.so动态库文件，并且该文件默认的名字是libapp.so，原因是Modules的名字是app
但是这里我们的LOCAL_MODULE用的是hello-jni，所以需要下一步
</pre>
8.在build.gradle中添加以下代码就可以将Modules的名字改掉
<pre>
========================================
ndk{
moduleName "hello-jni"
}
=======需要在defaultConfig中添加=========
</pre>
9.此时Run app还是提示错误，提示要在gradle.properties文件中添加</br>
android.useDeprecatedNdk=true  </br>
10.最后要在MainActivity中添加静态方法
<pre>
===================================================
static{
System.loadlibrary("hello-jni");
}
===================================================
</pre>
这样在app\build\intermediateds\ndk\debug\lib中生成libhello-jni.so文件了！
