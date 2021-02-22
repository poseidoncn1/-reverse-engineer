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
  - 字段1 dex_magic ,表示dex文件的文件表示，特征字符串
  - 字段2 checksum，表示校验和，对文件求32位的hash值(从字段3到文件末尾)
  - 字段3 signature[] ,表示SHA1，对文件求hash值(从字段4到文件末尾)
  - 字段4 file_size,表示文件大小
  ```
  字段2和字段3 就是确保dex 完整。改变dex，需更新这两个字段
  ```
  - 字段5 dex文件头大小
  - 字段6: 数据排列方式-小端方式

```
dex 文件 or pe文件数据格式都是小端方式保存；网络传输都是大端方式传输，使用小端主要原因是为了保存数据类型
```
- 各种数据的数组，包括字符串、类型、方法原型、字段、方法
- 类数据
- 其它

### 010editor分析dex结构
```
破解基于java。反编译的时候 隐藏代码或者转移代码，dex基本文件格式，如何加载，安卓虚拟机如何运行的

```
