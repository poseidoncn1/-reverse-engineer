### 安卓 NDK开发

```
基于c和c++开发，
原因：
   安全：更难反编译，加壳
   运行效率很高，速度快
   跨平台 x86 arm架构，NDK提供了各种ABI（application Binary Interface）供你选择,创建成功NDK会生成多个文件，来对应不同的硬件架构
```



```
坑：一个版本一个NDK状态，Android studio 2.2 以上基本上大致一致
```



```
Android Studio2.2 or 以上版本使用NDK将C和C++代码编译到原生库，然后Gradle将原生库打包APK。
java代码可以通过java 原生接口 JNI框架，调用原生库的函数;
Android Studio编译元素库默认编译工具为CMake，以前时ndk-build,现在也支持；


```

- 编译调试JNI代码，需要以下组件
  - Android NDK套件，保证在Android程序中使用c和c++
  - CMake 一款编译工具,与Gradle使用，编译原生库
  - LLDB： LLDB 调试器时基于LLVM的编译器框架，代码的加密 混淆框架基于LLVM做的

```
CMake编译脚本：CMakeLists.txt CMake如何将原生源文件编译入库or 使用ndk-build并包含Android.mk,
```





- Android Stadio NDK项目文件

  - 只要.java文件中，方法类型时native的，都在c++文件中实现的。

    ```
    static {
        System.loadLibray("native-lib");
    }
    将.so库文件，加载到 进程中，通过此语句
    ```

    ​

  - c++文件中函数的名称组成，java+包名+类名+函数名(中间以下划线分隔，函数名是java文件中的方法名)

  - c++文件中函数导出，一般写extern "C" ，以c方式导出。不写以c++导出，这样会名称粉碎，这样不仅导出函数名，还导出函数的参数，函数返回类型都导出。

  - c++文件中函数，jstring 返回类型，jstring 指针，只要是j开头的，都是java类数据类型（是重新定义的类型）

  - c++文件中的函数，默认有两个参数，如果java文件中方法有参数，在两个参数后面加

    - 参数1：JNIEnv* env ，JNI环境指针，我们用到的Jni函数，都在这个指针中
    - 参数2：jobject /*this*/，java的类的对象this。如果static 就是 jclass ,猜测是java的类

  ```
  一个项目问题：发生项目NDK配置没有正确配置，File->project structrue->ndk 路径
  ```

  ​

- 数据类型
  java数据类型在c++中的数据类型的表示，分为两类：

  - 引用类型 在c++中以j开头，本质都是指针。stdint.h 中定义,例如 jstring
  - 基本数据类型，在c++中以j开头，本质上就是c++中的数据类型的重定义。jni.h中声明，例如 jint

- 常用函数(参考androidJNI 函数参考)

  - java 文件调用c++层函数

    ```
    boolean Ret = stringFromJNI();
    ```

    ​

  - c++文件调用java层 方法

    ```
    //1.获取java层类类型
    jclass jclass1 = env->GetObjectClass(thiz);
    参数1：thiz是java层类对象this

    //2.获取java层方法ID
    jmethodID jmethodId = env->GetMethodID(jclass1，“showtext”,"()V");
    参数1：类类型
    参数2：java层方法
    参数3：java层方法类型

    //3.调用java层方法
    env->CallVoidMethod(thiz,jmethodId);
    参数1：java层类对象
    参数2：java层方法ID
    ```

    ​
