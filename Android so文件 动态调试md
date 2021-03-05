

### Android so文件

- readelf 去查看ELF文件，主要表信息。
  readelf -h 查看文件头

  -S 查看Section header Table        描述so文件那块是数据 那块是程序代码     A->Alloc 分配内存

  -I 查看程序头表 Program Header Table    so文件加载到内存是什么状态   Load->加载到内存

  -s 查看符号表

  -r 查看重定位表

  -d 产看动态加载信息

- Android Root

  ```
  1.第三方，利用漏洞提权 使用工具  kingroot 360超级root
  2.自己为了研究某一个漏洞 提权代码 编译，修改提权
  3.自己编译安卓源码
  ```

- so文件执行顺序 _init->init1->init2->init3->JNI_OnLoad

  ```
  都是被系统调用的，分析android 源代码

  ```

  ​

- Android so文件动态调试

  ```
  1.上传IDA调试服务端 ，IDA_pro/dbgsrv/android_server
    adb push android_server /data/local/tmp/ands
    修改权限，测试运行 //
    chmod 777 ands
    
    ./ands -p23456
  2.安装APK
  adb install .\app-dang.apk
  adb install -t .\app-debug.apk //-t ：允许测试包
  adb shell am start -D -m com.blue.ndkpacakname/.MainAcitiviy //启动一个类，.MainActivity中.省略的包名(因为和前面的包名一致) -D 调试方式启动
  adb forward tcp:23456（本地设备） tcp:23456(安卓设备)

  3.IDA
  Debugger -> Attach -> Remote ARMLinux/Android debugger

  4.启动jdb
  jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=8700
  ```






- 分析Android 逆向程序

  ```
  静态分析：
  先到jadx or android killer中找到清单文件，找到入口类（application有没有 android:name 属性），确认包名。找到资源空间资源 layout中确认空间布局。到入口类，入口函数找到函数调用，有native 即调用了JNI，

  找so文件，使用readelf查看so文件动态区段 readelf -d libxxx.so，找到到INIT类型字段，有INI_ARRAy类型字段。

  so文件 拖进ida进行反编译，F5后，以dword_xxx开头的,代表4字节数据。以sub_xxx开头的代表子函数。如果碰到左侧偏移地址为棕色，即代表是指令，选中认为是函数的部分，按 ‘P’键，转换成为函数。 
  动态分析：
  在完成adb IDA连接成功后，Debugger->debbuger option ：打勾 Suspend on library load/unload 后，运行 jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=8700 命令，ida 弹出窗口 取消 取消。Debbuger->Moudle List->找到so文件 双击->找到 我们 要找的函数。F2 下断点 ，运行，ida 弹出窗口 取消 apply。
  右上角有寄存器值 R0 R1...(有小箭头可以点，查看存储数据)，（一般R0最后作为返回值返回）

  {IDA动态调试中，看到 是函数调用 ，但是解析不是函数， 点击进去，按 C，可以解析出来 }
  ```

  ​



- 注意事项

  ```
  插件
  plugins/findcrypt.plw 识别加密算法
  plugins/AutoSetToLocalAnsiCodePage.plw 分析中文字符串
  IDAPython (python2.7 32位) ，IDA python上的插件，Keypatch（keyStone汇编引擎插件本身，需要安卓keystone python模块，可以更改 arm x86 汇编语言）

  另一种，附加调试，打开目标文件的ida数据库，附加调试，这个动态调试会改掉数据库。而我们第一种，不会改


  导入jni.h 头文件，可以识别jni一些结构体类型。

  分析技巧：
   smali代码中打印日志，hook 结合分析，打印参数，返回值等。把安卓源码改变，自己刷机查看日志，也可以
   
   CODE16 代表当前代码是thumb指令（ARM指令优化），CODE32 代表当前ARM指令。（ARM指令是定长指令，都是32位） thumb 和ARM指令可以混搭。 IDA Alt+G设置标志位T ，0代表ARM  1代表thumb
  ```

  ​
