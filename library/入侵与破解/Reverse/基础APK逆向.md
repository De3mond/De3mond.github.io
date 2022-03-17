# 基础APK逆向

## 工具推荐
* 很多工具都可以逆向APK/Java程序，自己选择
    * 这里推荐Jadx，或者JEB Decompiler（资源自己找）

## APK 逆向

### 基础
1. 命名规范
    * 开发的 APK 包名规范正常情况下是域名的反写（比如com.shit.app）
    * APK的主代码在 MainActivity 中
        * ![0](amWiki/images/reverse/apk/0.png "0")
2. JVM
    * 由于 JVM 开源，而且 JVM 的指令比本机指令还少的特性，因此基于 Java 开发的程序特别容易被逆向，只要针对 Java 编译后的程序的字节码进行解析，就可以很容易分析出源程序的内容
        * ![1](amWiki/images/reverse/apk/1.png "1")
3. JNI
    * Java Native Interface 技术可以实现 Java 代码和本地程序的双向交互,通过 Java 的本地接口来调用本地 C/C++ 程序，从而实现一些功能
        * 此类题目会稍难，但是出题方向更容易涉及到
        * 出题很多是把验证函数放在 JNI 里，通过 System.loadLibrary() 函数来调用 APK 的 so 文件中的函数
    * 我们可以解压apk包后利用IDA去分析其 so 文件来发现 flag 的蛛丝马迹




