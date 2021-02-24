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
    参数3：...args 方法参数
    ```

  - c++文件调用java层 字段

    ```
    //1.获取java层类类型
    jclass jclass1 = env->GetObjectClass(thiz);
    参数1：thiz是java层类对象this

    //2.获取java层字段 ID
    jfieldID jfieldId = env->GetFieldID(jclass1,"mStr","Ljava/lang/String;");
    参数1：类类型
    参数2：字段名
    参数3：字段类型

    //3.修改字段
    env->SetObjectField(thiz,jfieldId,env-NewStringUTF("hello"));
    参数1：类对象
    参数2：字段ID
    参数3：java 一个对象
    ```

  - 动态注册c++函数，即动态关联c++函数和java层方法

    ```
    在so文件加载时，在JNI_OnLoad方法中注册
    extern "C" JNIEXPORT jint JNI_OnLoad(JavaVM*vm ，void*reserved){
    //1.获取
           JNIEnv *env;
           jint jret = vm->GetEnv((void**)&env,JNI_VERSION_1_6);
           if(jret !=JNI_OK){
               return JNI_ERR;
           }
           
           

    //2.动态注册，绑定
         //2.1获取类类型
         jclass jclass1 = env->FindClass("com/example/hellworld/MainActivity");
         参数1：包名+类名
         
         //2.2准备结构体数组
         const JNINativeMethod method = {"fun1","(Ljava/lang/String;Ljava/lang/String;)V"，
         （void*）__NativeCFun};
         参数1：java层函数名 字符串
         参数2：函数类型
         参数3：c 函数指针
         
         //2.3注册
         env->RegisterNatives(jclass1,&method,1);
         注册一个方法
         参数1：类类型
         参数2：结构体数组指针
         参数3：方法个数
         
         
          return JNI_VERSION_1_4;

    }
    ```

    ​

    ​

  - 关于函数的其它知识

    ```
    1.JNI函数语言转换方式：
    c语言（*env）->NewStringUTF(env,"hello");  （这种方式逆向的是会出现）
    c++  env->NewStringUTF("hello");

    2.有些函数中，*isCopy说明
    无需关心，传入NULL即可



    ```

    ```
    ida插件：AutoSetToLocalAnsiCodePage.plw 解决乱码问题

    .so 文件在ida中打开，到exports中找到c++文件的函数，解析查看c++函数
    ```

    ```
    逆向极其重要：
    .so文件加载时，最早运行的函数是init函数,之后运行初始化数组，这个数组中的函数声明 需要加 __attribute__((constructor)),之后运行JNI_OnLoad

    _init->init1->init3->init2->JNI_OnLoad



    extern "C" void _init(void){

    }
    void __attribute__((constructor)) init1(void){
        
    }

    void __attribute__((constructor)) init3(void){
        
    }

    void __attribute__((constructor)) init2(void){
        
    }

    IDA中，在exports可以找到 _init,在Ctrl+F 后找到_init_array.
    readelf工具：
    sdk->ndk->x.x->toolchain->x86_64-->prebuilt->windows-x86_64->bin
    老：sdk->ndk-bundle->toolchain->x86_64-->prebuilt->windows-x86_64->bin

    readelf -d x.x.so
    确认：init函数是否存在，init_array是否存在，存在可以看到偏移，在ida中，按J
    init
    init_array
    ```

    ​

    ​

    ​

    ​

    ​

    ​

    ​

    ​
