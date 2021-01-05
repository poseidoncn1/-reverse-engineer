
## android reverse engineering

### android reverse tool
```markdown
1.dex文件分析工具 
 - 反编译工具baksmali.jar
 - 回编译工具smali.jar
java -jar baksmali.jar d class.dex -o out
java -jar smali.jar a out -o class.dex

2.AndroidManifest 清单文件分析工具
 - AXMLPrinter2.jar
  java -jar AXMLPrinter2.jar AndroidManifest.xml
  
3.Androd 集成分析工具
  - apktool.jar(baksmali.jar smali.jar AXMLPrinter2.jar aapt) 
  - 增强版 shakaapktool.jar 反编译资源增强
  
  java -jar apktool.jar d android.apk
  
  java -jar apktool.jar b android -o out
  
  其它 -r 不解密资源
  
  
  4、签名工具
  -signapk.jar
  
 java -jar signapk.jar testkey.x509.pem testkey.pk8 update.apk update_sined.apk
  
 ```
### android reverse tool other

dex2jar jd-gui jeb jadx GDA 
