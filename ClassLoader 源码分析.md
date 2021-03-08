---
typora-root-url: images
typora-copy-images-to: images
---

### 分析ClassLoader源码

```
想要在内存中加载dex，但提供的函数结构中并没有这个功能，需要分析DexClassLoader的构造和LoadClass方法。
分析要关注dexpath，找到 ClassLoader源码如何实现加载。
```

- DexClassLoader源码分析

  ```
  xref: /libcore/dalvik/src/main/java/dalvik/system/DexClassLoader.java

  DexClassLoader的构造调用了父类BaseDexClassLoader的构造。
  BaseDexClassLoader的源码同样在android系统的源码中
  ```

  ![1615084372262](.\1615084372262.png)

  - BaseDexClassLoader

    ```
    xref: /libcore/dalvik/src/main/java/dalvik/system/BaseDexClassLoader.java

    baseClassLoader 调用父类ClassLoader构造  和DexPathList 类的构造
    ```

    ![1615084861602](.\\1615084861602.png)

  - DexPathList

    ```
    xref: /libcore/dalvik/src/main/java/dalvik/system/DexPathList.java

    ```

    ![1615085320040](.\1615085320040.png)

    ​

  - DexPathList.makeDexElemets

    ```
    makeDexElemets 方法：第一个参数是一个element数组，猜测存放的是多个文件全路径

    ```

    ![1615086342629](/1615086342629.png)

    ```
    大写常量
    ```

    ![1615086554113](/1615086554113.png)

  - loadDexFile方法

    ```
    调用DexFile类中 loadDex方法
    ```

    ![1615086747485](/1615086747485.png)

  - DexFile类

    ```
    xref: /libcore/dalvik/src/main/java/dalvik/system/DexFile.java
    loadDex调用 new DexFile方法，再调用openDexFile方法,在调用openDexFileNative方法(c++代码)
    ```

    ![1615086975729](/1615086975729.png)

    ![1615087031065](/1615087031065.png)
    ![1615087152676](/1615087152676.png)

  - 分析

    ```
    static void Dalvik_dalvik_system_DexFile_openDexFileNative(const u4* args,
    152    JValue* pResult)
    153{
    154    StringObject* sourceNameObj = (StringObject*) args[0];
    155    StringObject* outputNameObj = (StringObject*) args[1];
    156    DexOrJar* pDexOrJar = NULL;
    157    JarFile* pJarFile;
    158    RawDexFile* pRawDexFile;
    159    char* sourceName;
    160    char* outputName;
    161
    162    if (sourceNameObj == NULL) {
    163        dvmThrowNullPointerException("sourceName == null");
    164        RETURN_VOID();
    165    }
    166
    167    sourceName = dvmCreateCstrFromString(sourceNameObj);
    168    if (outputNameObj != NULL)
    169        outputName = dvmCreateCstrFromString(outputNameObj);
    170    else
    171        outputName = NULL;
    172
    173    /*
    174     * We have to deal with the possibility that somebody might try to
    175     * open one of our bootstrap class DEX files.  The set of dependencies
    176     * will be different, and hence the results of optimization might be
    177     * different, which means we'd actually need to have two versions of
    178     * the optimized DEX: one that only knows about part of the boot class
    179     * path, and one that knows about everything in it.  The latter might
    180     * optimize field/method accesses based on a class that appeared later
    181     * in the class path.
    182     *
    183     * We can't let the user-defined class loader open it and start using
    184     * the classes, since the optimized form of the code skips some of
    185     * the method and field resolution that we would ordinarily do, and
    186     * we'd have the wrong semantics.
    187     *
    188     * We have to reject attempts to manually open a DEX file from the boot
    189     * class path.  The easiest way to do this is by filename, which works
    190     * out because variations in name (e.g. "/system/framework/./ext.jar")
    191     * result in us hitting a different dalvik-cache entry.  It's also fine
    192     * if the caller specifies their own output file.
    193     */
    194    if (dvmClassPathContains(gDvm.bootClassPath, sourceName)) {
    195        ALOGW("Refusing to reopen boot DEX '%s'", sourceName);
    196        dvmThrowIOException(
    197            "Re-opening BOOTCLASSPATH DEX files is not allowed");
    198        free(sourceName);
    199        free(outputName);
    200        RETURN_VOID();
    201    }
    202
    203    /*
    204     * Try to open it directly as a DEX if the name ends with ".dex".
    205     * If that fails (or isn't tried in the first place), try it as a
    206     * Zip with a "classes.dex" inside.
    207     */
    208    if (hasDexExtension(sourceName)//确认dex后缀，读取dex文件，pRawDexFile
    209            && dvmRawDexFileOpen(sourceName, outputName, &pRawDexFile, false) == 0) {
    210        ALOGV("Opening DEX file '%s' (DEX)", sourceName);
    211
    212        pDexOrJar = (DexOrJar*) malloc(sizeof(DexOrJar));// pDexOrJar申请结构体空间
    213        pDexOrJar->isDex = true;                         //初始化
    214        pDexOrJar->pRawDexFile = pRawDexFile;            //原有的File
    215        pDexOrJar->pDexMemory = NULL;
    216    } else if (dvmJarFileOpen(sourceName, outputName, &pJarFile, false) == 0) {
    217        ALOGV("Opening DEX file '%s' (Jar)", sourceName);
    218
    219        pDexOrJar = (DexOrJar*) malloc(sizeof(DexOrJar));
    220        pDexOrJar->isDex = false;
    221        pDexOrJar->pJarFile = pJarFile;
    222        pDexOrJar->pDexMemory = NULL;
    223    } else {
    224        ALOGV("Unable to open DEX file '%s'", sourceName);
    225        dvmThrowIOException("unable to open DEX file");
    226    }
    227
    228    if (pDexOrJar != NULL) {
    229        pDexOrJar->fileName = sourceName;
    230        addToDexFileTable(pDexOrJar);
    231    } else {
    232        free(sourceName);
    233    }
    234
    235    free(outputName);
    236    RETURN_PTR(pDexOrJar);
    237}
    238
    239/*
    240 * private static int openDexFile(byte[] fileContents) throws IOException
    241 *
    242 * Open a DEX file represented in a byte[], returning a pointer to our
    243 * internal data structure.
    244 *
    245 * The system will only perform "essential" optimizations on the given file.
    246 *
    247 * TODO: should be using "long" for a pointer.
    248 */
    249static void Dalvik_dalvik_system_DexFile_openDexFile_bytearray(const u4* args,
    250    JValue* pResult)
    251{
    252    ArrayObject* fileContentsObj = (ArrayObject*) args[0];//文件内容
    253    u4 length;
    254    u1* pBytes;
    255    RawDexFile* pRawDexFile;
    256    DexOrJar* pDexOrJar = NULL;
    257
    258    if (fileContentsObj == NULL) {
    259        dvmThrowNullPointerException("fileContents == null");
    260        RETURN_VOID();
    261    }
    262
    263    /* TODO: Avoid making a copy of the array. (note array *is* modified) */
    264    length = fileContentsObj->length;
    265    pBytes = (u1*) malloc(length);
    266
    267    if (pBytes == NULL) {
    268        dvmThrowRuntimeException("unable to allocate DEX memory");
    269        RETURN_VOID();
    270    }
    271
    272    memcpy(pBytes, fileContentsObj->contents, length);//文件内容copy到pBytes
    273
    274    if (dvmRawDexFileOpenArray(pBytes, length, &pRawDexFile) != 0) {//内存打开dex文件
    275        ALOGV("Unable to open in-memory DEX file");
    276        free(pBytes);
    277        dvmThrowRuntimeException("unable to open in-memory DEX file");
    278        RETURN_VOID();
    279    }
    280
    281    ALOGV("Opening in-memory DEX");
    282    pDexOrJar = (DexOrJar*) malloc(sizeof(DexOrJar));//申请Dex文件空间
    283    pDexOrJar->isDex = true;
    284    pDexOrJar->pRawDexFile = pRawDexFile;//存放原有文件
    285    pDexOrJar->pDexMemory = pBytes;//字节
    286    pDexOrJar->fileName = strdup("<memory>"); // Needs to be free()able.
    287    addToDexFileTable(pDexOrJar);
    288
    289    RETURN_PTR(pDexOrJar);
    290}
    291
    ```

  - java层 ，最终，返回了一个ncookie
    ![1615087031065](/1615087031065.png)

- 分析DexClassLoader类的loadClass方法,如何使用这个方法

  ```
  DexClassLoader->父类 BaseClassLoader->父类 ClassLoader->loadClass方法
  xref: /libcore/libart/src/main/java/java/lang/ClassLoader.java

  ```

  ​

  ​

  ​

  - 找到loadClass
    ​

- ​
