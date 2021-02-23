最简单的dex文件

- 生成 dex文件
  java -jar smali.jar -o class.dex HelloWorld.smali  /  zip HelloWorld.zip classes.dex
- 连接安卓设备
  adb devices
- 上传压缩文件
  adb push Helloworld.zip /data/local
- 启动并设置入口类,安卓手机执行的出来
  adb shell dalvikvm -cp /data/local/HelloWorld.zip HelloWorld

dex文件分为三大块

- dex文件头
  - 字段1 dex_magic ,表示dex文件的文件表示，特征字符串
  - 字段2 checksum，表示校验和，对文件求32位的hash值(从字段3到文件末尾)
  - 字段3 signature[] ,表示SHA1，对文件求hash值(从字段4到文件末尾)
  - 字段4 file_size,表示文件大小
      字段2和字段3 就是确保dex 完整。改变dex，需更新这两个字段
  - 字段5 dex文件头大小
  - 字段6: 数据排列方式-小端方式
        dex 文件 or pe文件数据格式都是小端方式保存；网络传输都是大端方式传输，使用小端主要原因是为了保存数据类型
    
  - 各种表的大小以及偏移
    - string_ids_size 和string_ids_off,字符串表的大小和偏移
    - type_ids_size和 type_ids_off 类型表的大小和偏移
    - proto_ids_size 和proto_ids_off 原型表的大小和偏移
    - field_ids_size 和field_ids_off，字段表的大小和偏移
    - method_ids_size和method_ids_off 方法表的大小和偏移
    - class_defs_size 和 class_defs_off 类数据的大小和偏移



- 各种数据的数组，包括字符串、类型、方法原型、字段、方法
      dex 文件，例如，名称等字符串，用数组的方式保存到一起，使用的时候使用数组的索引。
  
  - 字符串表dex_string_ids
      字符串表项，是一个字符串数据的偏移string_data_off，偏移指向的是一个string_data 结构。
  
  string_data 结构中有两个字段：
  - 
    - 字段1：代表长度，数据类型是uleb128,变长的数据类型(1-5字节)
    - 字段2：存储数据，字符串以0结尾
  - 类型表dex_type_ids
      类型表表项，是一个索引值，存放的是字符串表数组下标。
      类型描述符包括基本数据类型的描述符和类类型的描述符。
      LHelloworld;是HelloWorld类的类描述符。
      
  - 原型表 （方法原型）
        原型表项存储的是函数原型的各部分描述信息。包括短类型(shorty_idx)、返回类型(return_type_idx)、参数的类型(parameters_off) 最终还是一个指向字符串表的数组下标
        注：字段为返回类型(return_type_idx)的值，是类型表中的索引，无双引号；短类型是字符串，有双引号，是字符串的索引
    
  - 字段表（类成员）
        字段表项中内容存储的是字段的信息。包括字段所在类(class_idx)、字段的类型(type_idx)、字段的名称(name_idx),class_idx是类型表中的索引，type_idx是类型表中的索引，字段名称的索引是字符串表的数组下标
    
  
  - 方法表（类方法）
        方法表项存储的是方法的信息，包括方法所在的类(class_idx)、方法的原型(proto_idx)、方法的名称(name_idx),其中class_idx是类型表中的索引,proto_idx是原型表的索引，方法名称是字符床表数组的下标
    
  
- 
- 类数据
      类数据也是一个数组，每一个元素就是一个类的相关信息。
  - 不重要的信息
        所在的类class_idx,父类superclass_idx,接口 interface_off,访问权限access_flags
        类名索引，访问flags，父类索引，接口偏移，源码索引，注解偏移，类数据偏移
  - 数据保存在，class_data
        这个结构存放fields字段和method方法
        对我们重要的是类方法查找：
        class_data->encoded_method method->code_item code
        ->insns_size 指令长度 
          insns[]  指令
        简单分析一些指令转smali代码：需要查dalvik操作码
        1、6200 0000 -> sget-object v0,field[0]  //字段表索引0
        sget-object v0,out
        名称：out
        类型：java.io.PrintStream
        所在类：java.lang.System
        
        smali: sget-object v0,Ljava/lang/System;->out:Ljava.io.PrintStream;
        2、1A01 0000 const-string v1,string[0]  //字符串表索引0
        const-string v1,“Hello world”
        
        3、6E53 0600 0421  - invoke-virtual {v4,v0,v1,v2,v3},Test2.method5:(IIII)V;
        
    
  - 
  
  
- 其它
      http://androidxref.com 安卓源码下载







uleb128数据类型

- 特点：变长（1-5字节），每一个字节最高位表示标志位，可以理解为是否下一字节有效数据
- 范围：整型，最大表示一个32位的无符号数据
      整型数据 
      16进制 0x180 
      二进制：0000 0001 1000 0000->01 80
      小端方式二进制：1000 0000 0000 0001->80 01
      uleb128:1000 0000 0000 0011 ->80 03
      
      源码：
      
      DEX_INLINE int readUnsignedLeb128(const u1** pStream) {
          const u1* ptr = *pStream;
          int result = *(ptr++);
       
          if (result > 0x7f) {
              int cur = *(ptr++);
              result = (result & 0x7f) | ((cur & 0x7f) << 7);//第一个字节留7位，第二个字节留7位
                                                             //字节2会左移7位，与第一字节做 或操作
                                                             //举例  uleb128 1000 0000 0000 0011
                                                             //&0x7f 0000 0000 0000 0011
                                                             //A|(B<<7) 0000 0000 |(0000 0011<<7)
                                                             //         0000 0000 |1 1000 0000
                                                             //= 1 1000 0000=0x180
              if (cur > 0x7f) {
                  cur = *(ptr++);
                  result |= (cur & 0x7f) << 14;             //第3个字节，左移14位
                  if (cur > 0x7f) {
                      cur = *(ptr++);
                      result |= (cur & 0x7f) << 21;         //第4个字节，左移21位
                      if (cur > 0x7f) {
                          cur = *(ptr++);
                          result |= cur << 28;              //第5个字节，左移28位
                      }
                  }
              }
          }
       
          *pStream = ptr;
          return result;
      }
      
      结果就是一个整型数据：4字节
  

010editor分析dex结构

    破解基于java。反编译的时候 隐藏代码或者转移代码，dex基本文件格式，如何加载，安卓虚拟机如何运行的
    
