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

创建.XposedMain类实现IXposedHookLoadPackage方法
```markdown

class XposedMain implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {

        //xPosed Hook
        //包：om.bluelesson.helloapp//.MainActivity
        //类：om.bluelesson.helloappin.MainActivity$-1
        //函数： public void onClick(View paramAnonymousView)
    }
}
```
