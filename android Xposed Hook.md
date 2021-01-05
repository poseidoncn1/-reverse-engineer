## android Xposed Hook 

Xposed框架是一款开源java层的HOOK框架,可以hook java层任意方法,每次Hook成功后,需要重启设备.本文只记录个人学习过程.

### 安装Xposed

0x01 将XposedBridge-82.jar文件copy到project->app->lib目录下，并设置lib\XposedBridge-82.jar (provided 仅编译)  or 在build.gradle文件中 dependencies->添加 provided files('lib\\XposedBridge-82.jar')

0x02 安装android.xposed.installer

0x03 创建project->app->src->assets目录下，文件xposed_init(内容Xposed入口类) 默认使用 [com.example.lenovo.services.XposedMain]


0x04 AndroidMinfest.xml文件设置
```markdown
        <!--        使xposed模块有效 -->
        <meta-data android:name="xposedmodule" android:value="true"></meta-data>
        <!--        Xposed框架中模块列表显示的名字 -->
        <meta-data android:name="xposeddescription" android:value="xposedhook"></meta-data>
        <!--       Xposed框架支持的最低版本 -->
        <meta-data android:name="xposedminversion" android:value="54"></meta-data>
```



### 使用Xposed
使用Xposed Hook 实现获取对象实例,截取调用方法

创建.XposedMain类实现IXposedHookLoadPackage方法
```markdown
public class XposedMain implements IXposedHookLoadPackage {
    protected  Object myContext;
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {

        //xPosed Hook
        //包：com.bluelesson.helloapp    //入口类MainActivity
        //类：com.bluelesson.helloapp.MainActivity$1   //内部匿名类
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
