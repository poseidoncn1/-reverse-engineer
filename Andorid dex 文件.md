### 最简单的dex文件
- 生成 dex文件
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
   - 各种表的大小以及偏移
      - string_ids_size 和string_ids_off,字符串表的大小和偏移
      - type_ids_size和 type_ids_off 类型表的大小和偏移
      - proto_ids_size 和proto_ids_off 原型表的大小和偏移
      - field_ids_size 和field_ids_off，字段表的大小和偏移
      - method_ids_size和method_ids_off 方法表的大小和偏移
      - class_defs_size 和 class_defs_off 类数据的大小和偏移
- 各种数据的数组，包括字符串、类型、方法原型、字段、方法
```
dex 文件，例如，名称等字符串，用数组的方式保存到一起，使用的时候使用数组的索引。
```
   - 字符串表dex_string_ids
      ```
      字符串表项，是一个字符串数据的偏移string_data_off，偏移指向的是一个string_data 结构。
      ```
      string_data 结构中有两个字段：
      - 字段1：代表长度，数据类型是uleb128,变长的数据类型(1-5字节)
      - 字段2：存储数据，字符串以0结尾
   - 类型表dex_type_ids
     ```
     类型数组中，存放的是字符串表数组下标。
     ```

- 类数据
- 其它

### 010editor分析dex结构
```
破解基于java。反编译的时候 隐藏代码或者转移代码，dex基本文件格式，如何加载，安卓虚拟机如何运行的

```
