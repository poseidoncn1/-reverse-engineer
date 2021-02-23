### java层hook框架

```
hook 的本质是在某一个点拦截，做完我要做的事情，回到正确的轨道上继续运行
```



### Cydia Substrate  Apple->android 闭源

```
真机运行，必须root，ARM体系下，安装。
```

- 安装com.saurik.substrate 0.9.4010

- 编写Cydia Substrate 插件

  - 导入substrate-api.jar

    ```
    1、在app上右键，找到 打开模块设置(open Module Setting)
    2、选择 Dependencies项
        导入substrate-api.jar包
        implement
    ```

  - 设置权限和入口

    ```
    1、需要指的权限：cydia.permission.SUBSTRATE
    2、添加meta标签，name为cydia.permission.SUBSTRATE，value为下一步中创建的类名.Main
    <manifest xmlns:android="http">
    <application>
    <meta-data android:name="com.saurik.substrate.main" android:value=".Main"/>
    </application>
    <uses-permission android:name="cydia.permission.SUBSTRATE"/>
    </mainfest>

    ```

    ​

  - 新建一个类,实现回调函数

    ```
    1、新建类Main
    2、实现static void initialize()方法

    public class Main {
        static void initialize(){
            // ...插件加载时，指向代码
            //1.hook 类
            //包名：
            //类名 android.content.res.Resources;
            //方法原型：public int getColor(int id)
            
            MS.hookClassLoad(类名，new MS.ClassLoadHook(){
                //2.hook 指的方法
            }
            ，old)；
        
            
            
            
        }
    }
    ```

    ​

  - Hook 加载类
    使用MS.hookClassLoad

    ```
    MS.hookClassLoad("android.content.res.Resources",new MS.ClassLoadHook(){
        public void classLoaded(Class<?> resources) {
            //...当类加载时执行
            Method getColor=null;
            try {
            //获取老的方法
            //java 反射机制，在运行的时，使用类类型获取类的方法、字段等信息
               // getColor = resources.getMethod("getColor",Integer.TYPE);
               getColor = resources.getMethod("getColor",int.class);
               
                
            } catch {
            log.d("log","something is wrong");
            
            }
            if(getColor !=null) {
                //老的方法指针(函数指针)，
                final MS.MethodPointer old = new MS.MethodPointer();
                MS.hookMethod(resources,getColor,new MS.MethodHook(){
                     //public Object invoked(老的方法，参数)
                    public Object invoked(Object resoures,Object ... args) throws Throwable{
                        //hook代码
                        
                        //1、调用老的方法(老函数)
                            // int color =(Integer)old.invoke(resources,args);
                            int color =(int)old.invoke(resources,args);
                        //2、更改返回值，返回
                        return color &~0x0000ff00 | 0x00ff0000;
                   
                },old);//old在这里返回
            }
        }
    }
    );
    ```

    ​

  - Hook方法
    使用MS.hookMethod

    ```
    hook老的方法：
    void hookMethod(Class,Method,MethodHook<Object,Object>,MethodPointer<Object,Object>)

    替换老的方法：
    void hookMethod(Class,Method,MethodAlteration<Object,Object>)
    ```

    ​

- 关键函数

  - MS.hookClassLoad

    ```
    参数1：包名+类名
    参数2：(匿名类对象,实现接口)MS.ClassLoadHook的一个实例，当这个类被加载的时候，它的classLoaded方法会被指向
    hookClassLoad(String name,MS.ClassLoadHook hook);
    ```

    ​

  - MS.hookMethod

    ```
    参数1：加载的目标类，为classLoaded传下来的类参数
    参数2：需要hook的方法
    参数3：((匿名类对象,实现接口))MS.MethodHook的一个实例，其包含的invoked方法会被调用，以代替member中的代码
    参数4：老的函数指针，可以为空
    hookMethod(Class _class,Member member,MS.MethodHook hook,MS.MethodPointer old)
    ```

    ​





## Xposed Hook  android->Apple 开源 

```

```



#### 安装Xposed

- 将XposedBridge-82.jar文件copy到android stadio 项目中project->app->lib目录下，并设置lib\XposedBridge-82.jar (provided 仅编译)  or 在build.gradle文件中 dependencies->添加 provided files('lib\\XposedBridge-82.jar')

- 安装android.xposed.installer  安装:adb install de.robv.android.xposed.installer...apk

  ```
  /system可写入
  正在复制app_process（app启动进程）-> 对这个进程进行hook修改，完成注入，加载插件等操作
  ```

  ​

-  创建project->app->src->assets目录下，文件xposed_init(内容Xposed入口类) 默认使用 [com.example.lenovo.services.XposedMain]

-  AndroidMinfest.xml文件设置

```markdown
        <!--        使xposed模块有效 -->
        <meta-data android:name="xposedmodule" android:value="true"></meta-data>
        <!--        Xposed框架中模块列表显示的名字 -->
        <meta-data android:name="xposeddescription" android:value="xposedhook"></meta-data>
        <!--       Xposed框架支持的最低版本 -->
        <meta-data android:name="xposedminversion" android:value="54"></meta-data>
```

#### 实现插件

- 插件一个空Activity的工程

  ```

  ```

  ​

- 设置清单文件信息

  ````
    设置清单文件
   <!--        使xposed模块有效 -->       
   <meta-data android:name="xposedmodule" android:value="true"></meta-data>        <!--        Xposed框架中模块列表显示的名字 -->      
   <meta-data android:name="xposeddescription" android:value="xposedhook"></meta-data>        <!--       Xposed框架支持的最低版本 -->      
   <meta-data android:name="xposedminversion" android:value="54"></meta-data>
  ````

  ​

- 导入Xposed jar包，并设置为provided(compile only)

- 创建一个主类并实现Xposed中的接口

  ```
  public class XposedMain implements IXposedHookLoadPackage {
  }
  ```

  ​

- 重新handleLoadPackage方法

  ```
  public class XposedMain implements IXposedHookLoadPackage {
      protected  Object myContext;
      @Override
      public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
      
      }
  ```

  ​

- 建立xposed_init文件外部声明主类

  ```
   创建project->app->src->assets目录下，创建文件xposed_init(内容Xposed入口类) 默认使用 [com.example.lenovo.services.XposedMain]
  ```

  ​

- 在HandleLoadPackage中完善hook代码

  ```
  //hook代码
  public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {

          //xPosed Hook
          //包：com.bluelesson.helloapp    //入口类MainActivity
          //类：com.bluelesson.helloapp.MainActivity$1   //内部匿名类
          //方法： public void onClick(View paramAnonymousView)

          //
          if(loadPackageParam.packageName.equals("com.bluelesson.helloapp"))
          {
          
          }
          
     }
  ```

  ​



#### 关键函数

```
Xposed插件的关键Hook方法

XposedHelpers.findAndHookMethod方法
所属类：XposedHelpers
重载：
findAndHookMethod(类类型，方法名，参数类型，...回调)
//1、确保这个类可以访问到（系统类）
findAndHookMethod(TelephonyManager.class,"getDeviceId",new XC_MethodReplacement())

findAndHookMethod(类名，类加载器，参数类型，...回调)
//1、Hook目标程序的一个类，
//2、类加载器，所有类都要加载到内存，保证可以调用（类加载器负责加载这些类）
findAndHookMethod(className,loadPackageParam.classLoader,"CheckRegister
",String.class,String.class,new XC_MethodHook()...)


回调函数：
   1、XC_MethodReplacement 替换原有的方法
   需要重写replaceHookedMethod方法
   Object replaceHookedMethod(MethodHookParam)
   
   
   
   2、XC_MethodHook  不替换 ，在执行原有的方法之前，在执行原有的方法之后
   需要重写beforeHookedMethod 与 afterHookedMethod方法
   void beforeHookedMethod(MethodHookParam param)
   void afterHookedMethod(MethodHookParam param)
设置返回值：param.setResult(true);
   
```





### 使用Xposed

创建.XposedMain类实现IXposedHookLoadPackage方法
```markdown
public class XposedMain implements IXposedHookLoadPackage {
    protected  Object myContext;
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {

        //xPosed Hook
        //包：com.bluelesson.helloapp    //入口类MainActivity
        //类：com.bluelesson.helloapp.MainActivity$1   //内部匿名类，匿名类从1开始
        //函数： public void onClick(View paramAnonymousView)

        //
        if(loadPackageParam.packageName.equals("com.bluelesson.helloapp"))
        {
            Class cls  = XposedHelpers.findClass("com.bluelesson.helloapp.MainActivity",loadPackageParam.classLoader);

            //Hook构造方法没有参数，获取实例
            XposedHelpers.findAndHookConstructor(cls, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);

                }

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    super.afterHookedMethod(param);
                    myContext =  param.thisObject;
                    XposedBridge.log("11");
                }
            });

            //获取内部匿名类的方法
            Class cls01 = XposedHelpers.findClass("com.bluelesson.helloapp.MainActivity$1",loadPackageParam.classLoader);

            //Hook 匿名内部类的 onclick方法
            XposedHelpers.findAndHookMethod(cls01,"onClick", View.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                }

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    super.afterHookedMethod(param);
                    XposedBridge.log("22");
                    Toast.makeText((Context) myContext,"这是hook之后的内容",Toast.LENGTH_SHORT).show();

                    param.setResult(null);
                }
            });
        }

    }
}







      /*  class clz = XposedHelpers.findClass("",);
        XposedHelpers.findAndHookConstructor()*/
    }
}
```
