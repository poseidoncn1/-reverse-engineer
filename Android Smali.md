## Smali 语法

## smali 基础语法

### 基本数据类型
```markdown
  boolean Z
  byte  B
  short S
  char  C
  int I
  long  J
  float F
  void  V
```
### 引用类型
  - 对象
  Lpackagename/ObjectName;
  - 数组
  ```markdown
  [I  一维数组
  [[I 二维数组
  ```
### 字段
对象类型描述符->字段名:类型描述符
```markdown
Lpackagename/ObjectName;->myName:Lpackagename/String;
```
### 方法描述
```markdown
Lpackagename/ObjectName;->myFun(II[CI)V          ->void myFun(int,int,char [],int)  类packagename.ObjectName

```
### 局部变量
- .registers 指定方法中寄存器的数目
- .locals 表面方法非参寄存器的数目  

```markdown
寄存器传递参数默认规则如下：
1、当一个方法被调用的时候，方法的参数被置于最后N个寄存器中。如果一个方法有2个参数，5个寄存器(v0-v4),那么参数将置于最后2个寄存器 v3-v4
2、非静态方法中第一个参数总是调用该方法对象 

例子：
 非静态方法LMyObject;->callMe(II)V
 有2个整型参数，另外还有一个隐含的LMyObject;参数，所以总共有三个参数。假如指定了5个寄存器（v0-v4）,以.register方式指定5个 or 以.lcals方式指定2个。当方法被调用的时候，调用该方法的对象(即this)
 存放在v2中，第一个整型参数存放在v3中，第二个整型参数存放在v4中
 
寄存器命名方式：(baksmali默认使用P命名方式)
  V命名方式
  P命名方式
例子：5个寄存器 其中3个参数方法
     V命名方式 v0 v1 v2 v3 v4
     P命名方式 v0 v1 p0 p1 p2
                   this 参数 参数
                   
                   
 
 ```
 
 注意事项：J(long) or D(double) 需要2个寄存器   add-long/2add v2,p2 //v2=v2+p2  v3=v3+p3
