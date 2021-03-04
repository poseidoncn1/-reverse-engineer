# -reverse-engineer
### Android APK 加固

- APK加固介绍

  APK加固常用的方式，程序工程师

  - 使用proguard对APK中的源码进行混淆
    将源码的中的名称，进行重命名

    ```
    AndroidStudio默认已经有proguard的支持，在项目目录app下的build.gradle中有配置信息，只需将false变为true即可。
    buildTypes {
        release{
            minifyEnabled true
            proguardFiles...
        }
    }
    ```

    www.jianshu.com/p/60e82aafcfd0

  - 对Apk反编译之后的smali进行混淆
    对smali代码进行混淆

    ```
    代码乱序的原理：

    ```

    ​

  - 对Apk中的字符串进行加密

  - 对Apk中的文件进行校验

  ​

- 使用proguard 对APK中的源码进行混淆

- 使用各种方式对APK进行自我保护



### APK 加固

安全研究人员对APK加固采取的角度其实大致分为以下几方面：

1、将可执行代码dex文件加密，能够动态解密并执行

2、能够检测当前状态是否被调式，想尽一切办法反调试



### 加固知识点

- dex加载器

  - java 的ClassLoader 类加载器

    ```
    java中动态加载技术，程序运行在java虚拟机上，虚拟机需要将Class(将jar包中的Class)加载进来 创建实例，完成这一工作的就是ClassLoader
    ```

  - Android 的Dalvik/ART虚拟机 的 ClassLoader

    ```
    1、Boot类型的ClassLoader：Android类加载器不止一个，至少2个。创建一个Boot类型的ClassLoader实例，用来加载一些系统Framework层级需要的类，我们Android应用也需要系统的类，所以APP启动的时候也会把这个Boot类型的ClassLoader传进来。
    2、APP也有自己的类，保持在dex文件中，所以APP启动的时候，也会创建一个自己的ClassLoader实例，用于加载自己dex文件中的类。遍历ClassLoader

    ```

    ​

  - ClassLoader遍历 

    ​


    ClassLoader classloader = getClassLoader();
    if(classLoader != null){
        while(classLoader.getParent()!=null){
            classLoader =classLoader.getParent();
            Log.d(classLoader.toString())
        }
    }
    
    运行结果：
    一个是BootClassLoader（系统启动的时候创建）一个是PathClasLoader(应用启动的时候创建)
    PathClassLoader继承BootClassLoader 我们编写一个也要继承PathClassLoader
    

  - 动态加载实例知识点，遇到的问题
    动态加载外部的dex文件的时候，我们也可以使用自己创建的ClassLoader实例来加载dex里面的Class，创建一个ClassLoader，需要现有的ClassLoader实例作为 新实例的Parent。[这样整个Android系统所有的ClassLoader实例都会被一棵树关联起来，称为双亲代理模型 Parent Delegation Model]

    ```
    private ClassLoader(void unused,ClassLoader Parent){
        this.parent = parent;
    }
    ```

    双亲代理模型的特点和作用

    ```
    JVM中ClassLoader通过defineClass方法加载jar里面的Class而Android中 这个方法被废弃用了（但android底层其实并没有弃用）

    protected Class<?> loadClass(String name,boolean resolve){
        //首先检测类是否被加载
        Class c = findLoadedClass(name);
        if(c==null){//没有加载
            if(parent !=null){//父类加载
                c=parent.loadClass(name,false);
            }else{
                c=findBootstrapClassorNull(name);//系统获取
            }
            
            if(c==null){
            c=findClass(name);
            }
        }
        return c;
    }

    loadClass 方法：
    1、检查当前ClassLoader是否加载过此类，有就返回
    2、如果没有。查询Parent是否已经加载过此类，如果已经加载，返回父类加载的类。
    3、如果继承线上的ClassLoader都没有加载，才有Child执行类的加载。

    得出结论：
    如果一个类被位于树根的ClassLoader加载过，这个类永远不会被重新加载。

    ```

    使用ClassLoader需要注意：

    ```
    默认情况：
    如果通过动态加载，加载一个新版本的dex文件，使用里面的新类替换原有的旧类，从而修复原有类的BUG，那么你必须保证在加载新类的时候，旧类还没有被加载，因为如果已经加载过旧类，那么ClassLoader会一直优先使用旧类。

    如何新类旧类一起加载：
    由于ClassLoader总是会检查其Parent有没有加载过当前要加载的类，我们要使用一个和加载旧类的ClassLoader没有树继承关系的ClassLoader加载新类。

    新类旧类同时加载导致类型不符的异常情况：
    默认情况：
    在Java中，只有相同的ClassName+相同PackageName+相同ClassLoader，三者都相同，才会被认为是同一种类型。导致在实际使用中，可能会出现类型不符合的异常情况。

    ```

    ​

    ​

    ​

    ​

  - Android ClassLoader是抽象类具体子类->分为类加载器DexClassLoader和PathClassLoader

    ```
    DexClassLoader可以加载jar/apk/dex，可以从SD卡中加载未安装的APK
    PathClassLoader只能加系统中已经安装过的，
    ```




  - 安卓简单加固

    ```
    dex 生成条件：编译生成APK，将apk使用AndroidKiller反编译，将其中
    smali代码除了MyTestActivity.smali之外，全部删除，将只有MyTestActivity.smali的文件夹编译成dex

    1、copy dex到 asset目录下
    //获取assts目录管理器
    AssetManager as = getAssets();
    //合成路径
    String path = getFilesDir()+File.separator+dexName;
    //File.separator分隔符

    FileOutpuStream out = new FileOutpuStream(path);
    //打开文件
    InputStream is = as.open(dexName);
    //循环读取文件，拷贝到对应路径
    byte[] buffer = new byte[1024];
    int len = 0;
    while((len= is.read(buffer)!=-1){
        out.write(buffer,0,len);
    }
    out.close();

    2、load dex
    DexClassLoader dexClassLoader = new DexClassLoader(
      path, //dex路径
      getCacheDir().toString(),//优化之后的文件
      null,             //用到的原生库路径
      getClassLoader());//父类加载器
      
     
      
      3、调用类加载中的class 
      
      Class Test =  dexClassLoader.loadClass("包名.类名");
      //获取构造方法，无参构造
    Constructor localConStructor = TestDex.getConstructor(new Class[]{});//new Class[]{},参数 见java的反射机制
      //调用构造方法，创建对象
      Object instance = localConstructor.newInstance(new Object[]{});//new Object[]{} 等价于没有对象，可以省略不写。
      //获取成员方法
      Method methodTest = TestDex.getDeclaredMethod("createView",new Class[]{Activity.class});
      //取消java访问检查，提高反射速度
      methodTest.setAccessible(true);
      //调用方法，this指针传进去
      methodTest.invoke(instance,new Object[]{this}); //方法参数数组
      
      
      
    ```

    ​

  - ​

  - 替换PathClassLoader

    ```
    1、获取Activity类对象
    通过Public static ActivityThread currentActivityThread方法
    2、获取类对象获取成员变量mBoundApplication

    3、获取mBoundApplicatoin对象中的成员变量info


    4、获取info对象中的mClassLoader

    5.替换ClassLoader


    ```

    ​

  - ​

  - ​

    ````

    ````

    ​

    ​

    ​


- dex加固



### 初级加固

- 类加载器动态加载dex文件
  ​
- 设计傀儡dex替换原dex，动态加载dex
- 在傀儡dex中的各种地方添加反调试代码
