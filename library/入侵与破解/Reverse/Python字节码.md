# Python 字节码

-   虽然 Python 作为解释型语言，但是其也不是直接对源代码进行解释
    -   Python 解释器会将源代码处理成字节码后，借助 Python 解释器运行程序
-   通过 Python 自带的模块 dis 可以将目标函数转换成字节码

<!--more-->

```python
import dis

def fun(x, y, z):
    a = 1
    a += 1
    print("aaa")
    fun(1, 2, 3)
    return

dis.dis(fun)
```

-   控制台输出内容如下
    -   第一列对应的是源代码中的行号
    -   第二列对应的是源代码转化成的字节码
    -   第三列为此次操作对应的值（括号内为具体值）

-   -   
-   例如第六行
    -   解释器先读取了全局对象 `print` 函数，推入程序栈
    -   程序又将字符串 `'aaa'` 推入程序栈
    -   调用函数，并解释只有 1 个变量，解释器便会将栈顶的 1 个变量传递给函数，然后调用函数
        -   **需要注意，如果有多个参数的话，参数入栈顺序是从左到右，也就是最右边的参数在最顶端**
        -   `CALL_FUNCTION` 会在结束后弹出栈顶对应参数数量的元素，但是函数不会被弹出栈，因此最后有一个 `POP_TOP`

```bash
> python3 -u "/Users/biox/NutStore/Codes/VSCode/python/py1.py"
  4           0 LOAD_CONST               1 (1)
              2 STORE_FAST               3 (a)

  5           4 LOAD_FAST                3 (a)
              6 LOAD_CONST               1 (1)
              8 INPLACE_ADD
             10 STORE_FAST               3 (a)

  6          12 LOAD_GLOBAL              0 (print)
             14 LOAD_CONST               2 ('aaa')
             16 CALL_FUNCTION            1
             18 POP_TOP

  7          20 LOAD_GLOBAL              1 (fun)
             22 LOAD_CONST               1 (1)
             24 LOAD_CONST               3 (2)
             26 LOAD_CONST               4 (3)
             28 CALL_FUNCTION            3
             30 POP_TOP

  8          32 LOAD_CONST               0 (None)
             34 RETURN_VALUE
```

## 常见指令

-   详细内容见[官方文档](https://docs.python.org/zh-cn/3.6/library/dis.html#python-bytecode-instructions)

-   一般指令与一元操作指令

| 指令                | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| NOP                 | 无作用，用于占位                                             |
| POP_TOP             | 弹出栈顶元素                                                 |
| ROT_TWO             | 交换栈顶元素                                                 |
| LOAD_CONST          | 将读取的值推入栈                                             |
| LOAD_GLOBAL         | 将全局变量对象压入栈顶                                       |
| STORE_FAST          | 将栈顶指令存入对应局部变量                                   |
| COMPARE_OP          | 比较操作符                                                   |
| CALL_FUNCTION       | 调用函数                                                     |
| BUILD_SLICE         | 调用切片，跟的参数为切片的值的个数，一般从上到下为 [Val1:Val2:Val3] |
| JUMP_ABSOLUTE       | 向下跳转几句操作符，变量为跳转偏移量                         |
| UNARY_POSITIVE      | 实现 Val1 = +Val1                                            |
| UNARY_NEGATIVE      | 实现 Val1 = -Val1                                            |
| UNARY_NOT           | 实现 Val1 = not Val1                                         |
| UNARY_INVERT        | 实现 Val1 = ~Val                                             |
| FOR_ITER            | for 循环                                                     |
| GET_ITER            | 获取迭代器（一般后面跟循环）                                 |
| GET_YIELD_FROM_ITER | 获取 yield 生成器                                            |

-   二元操作指令

| 指令                   | 作用                                 |
| ---------------------- | ------------------------------------ |
| BINARY_POWER           | 乘方，栈顶数为指数                   |
| BINARY_MULTIPLY        | 乘法                                 |
| BINARY_MATRIX_MULTIPLY | 矩阵乘法，3.5 引入的新功能           |
| BINARY_FLOOR_DIVIDE    | 除法，结果向下取整                   |
| BINARY_TRUE_DIVIDE     | 除法                                 |
| BINARY_MODULO          | 取余                                 |
| BINARY_ADD             | 加法                                 |
| BINARY_SUBTRACT        | 减法                                 |
| BINARY_SUBSCR          | 数组取下标，栈顶为下标               |
| BINARY_LSHIFT          | 左移操作符（乘2）                    |
| BINARY_RSHIFT          | 右移操作符（除2向下取整）            |
| BINARY_AND             | 按位与                               |
| BINARY_XOR             | 异或                                 |
| BINARY_OR              | 按位或                               |
| STORE_SUBSCR           | 列表下标存储，例如 Val1[Val2] = Val3 |
| DELETE_SUBSCR          | 按下标删除元素，例如 del Val1[Val2]  |

-   自身操作指令，类似 `b += 1` ，就是上方有 BINARY 的指令的 BINARY 改成 INPLACE

-   其他指令见[官方文档](https://docs.python.org/zh-cn/3.6/library/dis.html#python-bytecode-instructions)

# Pyc 文件解析

-   Pyc 文件是 PythonCodeObject 对象的持久化保存方式
-   有时候会见到 Pyo 文件，这个是经过 Python 解释器优化后生成的字节码
    -   这个优化只是缩小了文件的体积，在代码运行速度上和 Pyc 差不多
-   尤其对于被 import 的文件，Python 解释器为了加快下一次被引用文件的读取速度，都会生成一个对应的 Pyc 文件
    -   当后续被 import 的时候，解释器会优先寻找持久化存储的对象
-   在 Python 源代码运行的时候，Python 解释器会先将代码处理成 PythonCodeObject 对象，保存在内存中处理
-   **除去预处理 PythonCodeObject 对象的过程，在执行速度上 Pyc 文件、Pyo 文件和源代码文件的速度相差无几**
-   **需要注意的是，Pyc 文件只能运行在生成出此文件的解释器版本上**
    -   **Python 在生成 Pyc 文件的时候也引入了 MagicNumber，来标示此 Pyc 文件对应的版本号**
-   在 Python 解释器目录下 `./lib/python3.7/importlib/_bootstrap-external.py` 中有明确的版本号记录
    -   这里的版本号是解释器字节码更新的版本号

```
# Magic word to reject .pyc files generated by other Python versions.
# It should change for each incompatible change to the bytecode.
#
# The value of CR and LF is incorporated so if you ever read or write
# a .pyc file in text mode the magic number will be wrong; also, the
# Apple MPW compiler swaps their values, botching string constants.
#
# There were a variety of old schemes for setting the magic number.
# The current working scheme is to increment the previous value by
# 10.
#
# Starting with the adoption of PEP 3147 in Python 3.2, every bump in magic
# number also includes a new "magic tag", i.e. a human readable string used
# to represent the magic number in __pycache__ directories.  When you change
# the magic number, you must also set a new unique magic tag.  Generally this
# can be named after the Python major version of the magic number bump, but
# it can really be anything, as long as it's different than anything else
# that's come before.  The tags are included in the following table, starting
# with Python 3.2a0.
#
# Known values:
#  Python 1.5:   20121
#  Python 1.5.1: 20121
#     Python 1.5.2: 20121
#     Python 1.6:   50428
#     Python 2.0:   50823
#     Python 2.0.1: 50823
#     Python 2.1:   60202
#     Python 2.1.1: 60202
#     Python 2.1.2: 60202
#     Python 2.2:   60717
#     Python 2.3a0: 62011
#     Python 2.3a0: 62021
#     Python 2.3a0: 62011 (!)
#     Python 2.4a0: 62041
#     Python 2.4a3: 62051
#     Python 2.4b1: 62061
#     Python 2.5a0: 62071
#     Python 2.5a0: 62081 (ast-branch)
#     Python 2.5a0: 62091 (with)
#     Python 2.5a0: 62092 (changed WITH_CLEANUP opcode)
#     Python 2.5b3: 62101 (fix wrong code: for x, in ...)
#     Python 2.5b3: 62111 (fix wrong code: x += yield)
#     Python 2.5c1: 62121 (fix wrong lnotab with for loops and
#                          storing constants that should have been removed)
#     Python 2.5c2: 62131 (fix wrong code: for x, in ... in listcomp/genexp)
#     Python 2.6a0: 62151 (peephole optimizations and STORE_MAP opcode)
#     Python 2.6a1: 62161 (WITH_CLEANUP optimization)
#     Python 2.7a0: 62171 (optimize list comprehensions/change LIST_APPEND)
#     Python 2.7a0: 62181 (optimize conditional branches:
#                          introduce POP_JUMP_IF_FALSE and POP_JUMP_IF_TRUE)
#     Python 2.7a0  62191 (introduce SETUP_WITH)
#     Python 2.7a0  62201 (introduce BUILD_SET)
#     Python 2.7a0  62211 (introduce MAP_ADD and SET_ADD)
#     Python 3000:   3000
#                    3010 (removed UNARY_CONVERT)
#                    3020 (added BUILD_SET)
#                    3030 (added keyword-only parameters)
#                    3040 (added signature annotations)
#                    3050 (print becomes a function)
#                    3060 (PEP 3115 metaclass syntax)
#                    3061 (string literals become unicode)
#                    3071 (PEP 3109 raise changes)
#                    3081 (PEP 3137 make __file__ and __name__ unicode)
#                    3091 (kill str8 interning)
#                    3101 (merge from 2.6a0, see 62151)
#                    3103 (__file__ points to source file)
#     Python 3.0a4: 3111 (WITH_CLEANUP optimization).
#     Python 3.0b1: 3131 (lexical exception stacking, including POP_EXCEPT
                          #3021)
#     Python 3.1a1: 3141 (optimize list, set and dict comprehensions:
#                         change LIST_APPEND and SET_ADD, add MAP_ADD #2183)
#     Python 3.1a1: 3151 (optimize conditional branches:
#                         introduce POP_JUMP_IF_FALSE and POP_JUMP_IF_TRUE
                          #4715)
#     Python 3.2a1: 3160 (add SETUP_WITH #6101)
#                   tag: cpython-32
#     Python 3.2a2: 3170 (add DUP_TOP_TWO, remove DUP_TOPX and ROT_FOUR #9225)
#                   tag: cpython-32
#     Python 3.2a3  3180 (add DELETE_DEREF #4617)
#     Python 3.3a1  3190 (__class__ super closure changed)
#     Python 3.3a1  3200 (PEP 3155 __qualname__ added #13448)
#     Python 3.3a1  3210 (added size modulo 2**32 to the pyc header #13645)
#     Python 3.3a2  3220 (changed PEP 380 implementation #14230)
#     Python 3.3a4  3230 (revert changes to implicit __class__ closure #14857)
#     Python 3.4a1  3250 (evaluate positional default arguments before
#                        keyword-only defaults #16967)
#     Python 3.4a1  3260 (add LOAD_CLASSDEREF; allow locals of class to override
#                        free vars #17853)
#     Python 3.4a1  3270 (various tweaks to the __class__ closure #12370)
#     Python 3.4a1  3280 (remove implicit class argument)
#     Python 3.4a4  3290 (changes to __qualname__ computation #19301)
#     Python 3.4a4  3300 (more changes to __qualname__ computation #19301)
#     Python 3.4rc2 3310 (alter __qualname__ computation #20625)
#     Python 3.5a1  3320 (PEP 465: Matrix multiplication operator #21176)
#     Python 3.5b1  3330 (PEP 448: Additional Unpacking Generalizations #2292)
#     Python 3.5b2  3340 (fix dictionary display evaluation order #11205)
#     Python 3.5b3  3350 (add GET_YIELD_FROM_ITER opcode #24400)
#     Python 3.5.2  3351 (fix BUILD_MAP_UNPACK_WITH_CALL opcode #27286)
#     Python 3.6a0  3360 (add FORMAT_VALUE opcode #25483)
#     Python 3.6a1  3361 (lineno delta of code.co_lnotab becomes signed #26107)
#     Python 3.6a2  3370 (16 bit wordcode #26647)
#     Python 3.6a2  3371 (add BUILD_CONST_KEY_MAP opcode #27140)
#     Python 3.6a2  3372 (MAKE_FUNCTION simplification, remove MAKE_CLOSURE
#                         #27095)
#     Python 3.6b1  3373 (add BUILD_STRING opcode #27078)
#     Python 3.6b1  3375 (add SETUP_ANNOTATIONS and STORE_ANNOTATION opcodes
#                         #27985)
#     Python 3.6b1  3376 (simplify CALL_FUNCTIONs & BUILD_MAP_UNPACK_WITH_CALL
                          #27213)
#     Python 3.6b1  3377 (set __class__ cell from type.__new__ #23722)
#     Python 3.6b2  3378 (add BUILD_TUPLE_UNPACK_WITH_CALL #28257)
#     Python 3.6rc1 3379 (more thorough __class__ validation #23722)
#     Python 3.7a1  3390 (add LOAD_METHOD and CALL_METHOD opcodes #26110)
#     Python 3.7a2  3391 (update GET_AITER #31709)
#     Python 3.7a4  3392 (PEP 552: Deterministic pycs #31650)
#     Python 3.7b1  3393 (remove STORE_ANNOTATION opcode #32550)
#     Python 3.7b5  3394 (restored docstring as the first stmt in the body;
#                         this might affected the first line number #32911)
#
# MAGIC must change whenever the bytecode emitted by the compiler may no
# longer be understood by older implementations of the eval loop (usually
# due to the addition of new opcodes).
#
# Whenever MAGIC_NUMBER is changed, the ranges in the magic_values array
# in PC/launcher.c must also be updated.
```

-   对于 pyc 文件整体的 C 结构体，可以在 `./include/python2.7/code.h` 或不同版本类似的文件中找到
    -   具体代码如下

```C
/* Bytecode object */
typedef struct {
    PyObject_HEAD
    int co_argcount;		/* #arguments, except *args */
    int co_nlocals;		/* #local variables */
    int co_stacksize;		/* #entries needed for evaluation stack */
    int co_flags;		/* CO_..., see below */
    PyObject *co_code;		/* instruction opcodes */
    PyObject *co_consts;	/* list (constants used) */
    PyObject *co_names;		/* list of strings (names used) */
    PyObject *co_varnames;	/* tuple of strings (local variable names) */
    PyObject *co_freevars;	/* tuple of strings (free variable names) */
    PyObject *co_cellvars;      /* tuple of strings (cell variable names) */
    /* The rest doesn't count for hash/cmp */
    PyObject *co_filename;	/* string (where it was loaded from) */
    PyObject *co_name;		/* string (name, for reference) */
    int co_firstlineno;		/* first source line number */
    PyObject *co_lnotab;	/* string (encoding addr<->lineno mapping) See
				   Objects/lnotab_notes.txt for details. */
    void *co_zombieframe;     /* for optimization only (see frameobject.c) */
    PyObject *co_weakreflist;   /* to support weakrefs to code objects */
} PyCodeObject;
```

-   将 Python 源代码生成为 Pyc 文件，这里使用的版本是 Python 2.7.6
    -   这里我们将一个名为 `py1.py` 的文件

```python
import py_compile
py_compile.compile('./py1.py')
```

-   会在源代码文件目录下找到编译后的文件
-   使用 010 editor 打开，会提示是否需要安装 pyc 字节码辅助插件，虽然说只支持 2.4 - 2.7
    -   Python 3 以后的都不能用...

![][1]

-	此处以中南大学2020校赛的一道逆向题目 [py&flower.pyc](https://github.com/CSUAuroraLab/ACTF2020/raw/master/Reverse/%E8%9B%87%E4%B8%8E%E8%8A%B1/release/py%26flower.pyc) 作为介绍，使用 010 editor 打开

![][2]

-   最前面的 4 个字节为 Magic Number ，其中前两个直接为解释器的版本号
    -   此处前两个字节为 62211，也就是 Python 2.7.0a0 版本的字节码解释器
    -   注意这里是小端序，就是高位在后面，所以是 0xF303

-   Magic Number 之后的四个字节为时间戳，这里是 0x5EC652B0，之后就是 Python 代码对象
-   代码对象首先一个字节表示此处的对象类型，这里值为 TYPE_CODE，值为 0x63，
-   此后四个字节表示参数的个数，也就是 co_argcount 的值
-   往后四个字节是局部变量的个数 co_nlocals
-   往后四个字节是栈空间大小 co_stacksize
-   往后四个字节是 co_flags 
-   之后就是 co_code 了，也就是编译好的字节码的部分
    -   co_code 部分首先的一个字节也是表示此处的对象类型，这里是 TYPE_STRING，为 0x73
    -   接下来四个字节表示此 co_code 对象的长度，此后就是代码对象，这里的代码长度为 0xA7
    -   也就是后方 163 个字节的长度都是代码对象

![][3]

-   此 co_code 对象的字节码内容结束后，接着是 co_consts 内容，也就是用到的常量的内容
    -   最开始是 TYPE_TUPLE，表示这是个元组类型
    -   此后四个字节是元素个数，这里是 0x23，之后每一个字节与对应的值一组，一共 0x23 组
        -   每组中第一个字节表示元素类型，比如 0x69 指 TYPE_INT，此后为对应的值
-   后方也对应结构体中的相应内容

![][4]

# 字节码混淆

## Anti-uncompyle6

-   对于正常的 pyc 文件，使用 uncompyle6 插件可以正常的进行字节码逆向，得到原来的代码

```bash
> uncompyle6 ./py2.pyc
```

-   如果需要使 uncompyle6 失效的话，只要在 co_code 头部加上 `0x71 0x03 0x00` ，然后把记录 co_code 长度的数据加 3
    -   这段字节码指 `JUMP_ABSOLUTE 3` ，也就是向后跳 3 个字节后继续执行，实际上没有改变代码逻辑
    -   但是 uncompyle6 插件的还原逻辑就没办法识别此字节码原先的意思，导致解析异常

## Anti-dis

-   上文的改法会导致 uncompyle6 插件异常，但是这个方法的实质只是增加了一句字节码
-   Python 可以借助自带的 dis 库和 marshal 库解析 pyc 二进制文件中的信息，此处以一个简单的代码作为例子

```python
def fun1():
    enc = "Ua`|{f.4V}$l4h4Vx{s.4|``dg.;;vx{s:v}$l:wz;4h4Dxqugq4}zp}wu`q4`|q4g{afwq4ur`qf4m{af4pqf}bu`}{z"
    flag = ""
    for i in enc:
        flag += chr(ord(i) ^ 0x14)
    print flag
fun1()
```

-   编译成 pyc 文件后，尝试加入 `JUMP_ABSOLUTE 3` 到代码头部
    -   橙色的字节为编辑过的

![][5]

-	使用 uncompyle6 发生 Parse error 异常，但是还是可以正常运行

![][6]

-   尝试使用 marshal 模块搭配 dis 模块进行字节码解析

```python
import marshal, dis
fp = open("./py1.pyc")
fp.read(8) # Read out magic number and timestamp
co_code = marshal.load(fp)
dis.dis(co_code)
```

-   程序输出了完整的字节码，根据字节码还是可以顺利的还原出源代码信息
    -   可以发现，头部已经加上了我们自己编辑的 `JUMP_ABSOLUTE 3`

```bash
> python2 -u "/Users/biox/NutStore/Codes/VSCode/python/py2.py"
  1           0 JUMP_ABSOLUTE            3
        >>    3 LOAD_CONST               0 (<code object fun1 at 0x1010486b0, file "./py1.py", line 1>)
              6 MAKE_FUNCTION            0

  7           9 STORE_NAME               0 (fun1)
             12 LOAD_NAME                0 (fun1)
             15 CALL_FUNCTION            0
             18 POP_TOP             
             19 LOAD_CONST               1 (None)
             22 RETURN_VALUE  
```

-   如果我们不想让 dis 顺利的导出字节码，也可以用一些指令来使得 dis 模块产生异常
    -   比如来个指令重叠，中间插一个读取数据的字节码`0x71 0x04 0x00 0x64 0x71 0x08 0x00 0x00`
    -   这里的 `0x64` 为解释器的 LOAD_CONST 指令，如果正常的话这里应该是 `LOAD_CONST 0x0871`
    -   那么 dis 模块就看不懂了，实际上通过前面的 `0x71 0x04 0x00` 会跳过此字节码，实际上是不执行的
    -   后方的`0x71 0x08 0x00` 是根据前面第一个 `0x71` 开始跳转的，所以跟的是 `0x08`

![][7]

-   尝试 dis 字节码，直接抛出 IndexError 了，同时 uncompyle6 也 IndexError 了

```bash
> python2 -u "/Users/biox/NutStore/Codes/VSCode/python/py2.py"                                                                        python
  1           0 JUMP_ABSOLUTE            4
              3 LOAD_CONST            2161
Traceback (most recent call last):
  File "/Users/biox/NutStore/Codes/VSCode/python/py2.py", line 5, in <module>
    dis.dis(co_code)
  File "/usr/local/Cellar/python@2/2.7.16_1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/dis.py", line 43, in dis
    disassemble(x)
  File "/usr/local/Cellar/python@2/2.7.16_1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/dis.py", line 98, in disassemble
    print '(' + repr(co.co_consts[oparg]) + ')',
IndexError: tuple index out of range
```

-   如果需要还原成能正常 dis 的 pyc 文件，只能手动修补了

# 动态创建内置类型

-   Python 还有一个自带的库，叫做 types，借助这个库可以生成 Python 内置的类型
-   比如生成 code 对象，这里以 2020 年 XCTF-CyBRICS 逆向 Ployglot 为例，其最后一层逆向的 Python 代码就是用了此方法

```python
import types

def define_func(argcount, nlocals, code, consts, names):
    #PYTHON3.8!!!
    def inner():
        return 0

    fn_code = inner.__code__
    cd_new = types.CodeType(argcount,
                             0,
                             fn_code.co_kwonlyargcount,
                             nlocals,
                             1024,
                             fn_code.co_flags,
                             code,
                             consts,
                             names,
                             tuple(["v%d" for i in range(nlocals)]),
                             fn_code.co_filename,
                             fn_code.co_name,
                             fn_code.co_firstlineno,
                             fn_code.co_lnotab,
                             fn_code.co_freevars,
                             fn_code.co_cellvars)
    inner.__code__ = cd_new
    return inner

f1 = define_func(2,2,b'|\x00|\x01k\x02S\x00', (None,), ())
f2 = define_func(1,1,b't\x00|\x00\x83\x01S\x00', (None,), ('ord',))
f3 = define_func(0,0,b't\x00d\x01\x83\x01S\x00', (None,  'Give me flag: '), ('input',))
f4 = define_func(1, 3, b'd\x01d\x02d\x03d\x04d\x05d\x01d\x06d\x07d\x08d\td\x03d\nd\x0bd\x0cd\rd\x08d\x0cd\x0ed\x0cd\x0fd\x0ed\x10d\x11d\td\x12d\x03d\x10d\x03d\x0ed\x13d\x0bd\nd\x14d\x08d\x13d\x01d\x01d\nd\td\x01d\x12d\x0bd\x10d\x0fd\x14d\x03d\x0bd\x15d\x16g1}\x01t\x00|\x00\x83\x01t\x00|\x01\x83\x01k\x03r\x82t\x01d\x17\x83\x01\x01\x00d\x18S\x00t\x02|\x00|\x01\x83\x02D\x00]$}\x02t\x03|\x02d\x19\x19\x00t\x04|\x02d\x1a\x19\x00\x83\x01\x83\x02d\x18k\x02r\x8c\x01\x00d\x18S\x00q\x8cd\x1bS\x00',
                 (None, 99, 121, 98, 114, 105, 115, 123, 52, 97, 100, 51, 101, 55, 57, 53, 54, 48, 49, 50, 56, 102, 125, 'Length mismatch!', False, 1, 0, True),
                 ('len', 'print', 'zip', 'f1', 'f2'))
f5 = define_func(0, 1,b't\x00\x83\x00}\x00t\x01|\x00\x83\x01d\x01k\x08r\x1ct\x02d\x02\x83\x01\x01\x00n\x08t\x02d\x03\x83\x01\x01\x00d\x00S\x00',(None, False, 'Nope!', 'Yep!'), ('f3', 'f4', 'print'))
f5()
```

-   使用给定的字节码，构造 CodeType 对象，直接转换成函数来调用
-   只要把对应的 PyCodeObject 中应该有的值构造正确，就能顺利执行
-   这样的函数如果没加花指令，是可以直接 dis 出来的


[1]: https://www.bi0x.cn/upload/2021/06/2131804168-34588c448dc34572a60ce87d3e1227fd.png
[2]: https://www.bi0x.cn/upload/2021/06/3067244573-5ffe6e1232a64d67af5370d9a7519d4c.png
[3]: https://www.bi0x.cn/upload/2021/06/1781230388-d512fd62e2cf478ba07493d740d69a41.png
[4]: https://www.bi0x.cn/upload/2021/06/1632998359-6c6d93606884482bb7021f4c82c892f0.png.png
[5]: https://www.bi0x.cn/upload/2021/06/1770849281-63ccc3337e994fdcb721f0e7b86e5a98.png
[6]: https://www.bi0x.cn/upload/2021/06/2557337332-cf1861aef0f049b5afec656c42ab1506.png
[7]: https://www.bi0x.cn/upload/2021/06/171434736-a8ac1737e31d437a9f86ffa06706cc12.png