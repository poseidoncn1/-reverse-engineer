https://blog.csdn.net/qiming_zhang/article/details/7309909#3.2.4

### PE文件格式
DOS头 0x5A4D;NT 头 0x00004550;Section;
### PE 文件后缀
ocx com sys exe dll
### 加载PE文件  or 实现 LoadLibrary dll
```
1.判断是否是PE文件 or dll文件
2.将各个section内存对齐加载到内存中
3.修复重定位
4.初始化PE数据堆栈
5.填充IAT
6.执行tls回调函数，执行pe文件

```

### NT头中 文件头  可选头


- 可选头 数据目录表
  - 导出表
    函数名表、序号表、函数地址表  
    ```
    1、一般导入表损坏了，可以通过导出表修复，使用GetProcAddress 检测IAT HOOK)
    2、名称表没有是序号导出
    ```
    
  - 导入表
    Name(DllName)、INT、IAT
    ```
    1、IAT 和INT表一般情况下一致。INT可以不存在，IAT必须存在。否则文件找不到正确的函数地址PE文件无法运行；很多程序会隐藏IAT导入地址表，防止别人捕捉到PE文件调用了那些重要API函数信息。
    2、最高位为1 序号导入
    ```
  - 资源表
   资源种类的个数，此种类资源个数，这个资源RVA位置
   ```
   一般PE文件资源，存放很多重要的PE文件，释放资源后，可以作为独立的程序运行
   ```

  - 重定位表
    重定位表存放的是需要重定位地址的地址
    ```
    修复公式：1、需要重定位地址的地址：address=新加载基址+RVA+offset
             2、修改需要重定位地址 address =address-40 0000+新的加载基址 （如果是dll默认加载基址 1000 00000）
    ```
  - 
