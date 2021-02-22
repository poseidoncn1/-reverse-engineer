### 最简单的dex文件
-生成 dex文件
java -jar smali.jar -o class.dex HelloWorld.smali
- 连接安卓设备
adb devices
- 上传压缩文件
adb push Helloworld.zip /data/local
- 启动并设置入口类
adb shell dalvikvm -cp /data/local/HelloWorld.zip HelloWorld

### dex文件分为三大块
- dex文件头
  
- 各种数据的数组，包括字符串、类型、方法原型、字段、方法
- 类数据
- 其它


```
破解基于java。反编译的时候 隐藏代码或者转移代码，dex基本文件格式，如何加载，安卓虚拟机如何运行的

```
