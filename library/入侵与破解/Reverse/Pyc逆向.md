# Pyc逆向和Python的可执行程序（exe）逆向

## 基础Pyc逆向
1. 此类题目比较简单，只需要下载一个插件叫 uncompyle6 `pip install uncompyle6`就可以了
2. 使用方法是`uncompyle6 -o test.py test.pyc`,将 pyc 文件反编译写进 py 文件中
3. 再查看 Python 程序中的内容解出 flag ，通常此类题目都是结合比较复杂的密码题出的

## 深入解析
### Python字节码
* Python 虽然是一种解释型语言，但也不是对源代码直接进行解释的，其本质是将源代码进行翻译，翻译成 Python 解释器能理解的指令，这些指令就称为**字节码**
* Python 解释器运行代码的步骤实际上是
    1. 先将源代码编译成字节码文件，这个文件一般为 pyc 或者 pyo 文件
    2. 使用 Python 解释器读取此编译后的文件
    3. 解释器进行解释，实现代码的运行
* 如果要看一个函数的 Python 字节码，可以使用 dis 模块实现局部编译

```
                import dis
                def fun(x, y, z):
                    a = 1
                    a += 1
                    print("aaa")
                    fun(1, 2, 3)
                    return
                dis.dis(fun)
                    | |
                    | |
                    \ /

  3           0 LOAD_CONST               1 (1)
              2 STORE_FAST               3 (a)    

  4           4 LOAD_FAST                3 (a)    
              6 LOAD_CONST               1 (1)    
              8 INPLACE_ADD
             10 STORE_FAST               3 (a)    

  5          12 LOAD_GLOBAL              0 (print)
             14 LOAD_CONST               2 ('aaa')
             16 CALL_FUNCTION            1
             18 POP_TOP

  6          20 LOAD_GLOBAL              1 (fun)
             22 LOAD_CONST               1 (1)
             24 LOAD_CONST               3 (2)
             26 LOAD_CONST               4 (3)
             28 CALL_FUNCTION            3
             30 POP_TOP

  7          32 LOAD_CONST               0 (None)
             34 RETURN_VALUE
```
* 遇到只给字节码的题目的时候，没有什么工具可以帮助分析字节码,只能自行阅读字节码，然后翻译回原来的 Python 代码,之后的内容就是对 Python 代码进行逆向了
* 常见的 Python 字节码

|  **指令**   | **作用**  |
|  ----  | ----  |
| NOP | 无作用，用于占位 |
| POP_TOP | 弹出栈顶元素 |
| LOAD_CONST | 将读取的值推入栈 |
| LOAD_GLOBAL | 将全局变量对象压入栈顶 |
| STORE_FAST | 将栈顶指令存入对应局部变量 | 
|COMPARE_OP | 比较操作符 |
| CALL_FUNCTION | 调用函数 |
| BUILD_SLICE | 调用切片，跟的参数为切片的值的个数 |
| JUMP_ABSOLUTE | 向下跳转几句操作符，变量为跳转偏移量 |
| UNARY_POSITIVE | 实现Val1=+Val1 |
| UNARY_NEGATIVE | 实现Val1=-Val1 |
|UNARY_NOT | 实现 Vall = not Val1 |
|UNARY_INVERT | 实现 Val1 = -Val |
|FOR_ITER | for 循环 |
|GET_ITER | 获取迭代器（一般后面跟循环） |
|GET_YIELD_FRON_ITER | 获取 yield 生成器 |
| BINARY_POWER | 乘方，栈顶数为指数 |
|BINARY_MULTIPLY | 乘法 |
|BINARY_MATRIX_MULTIPLY | 矩阵乘法，3.5引入的新功能 |
|BINARY_FLOOR_DIVIDE | 除法，结果向下取整 |
|BINARY_TRUE_DIVIDE | 除法 |
|BINARY_LMODULO | 取余 |
|BINARY_ADD | 加法 |
|BINARY_SUBTRACT | 减法 |
|BINARY_SUBSCR | 数组取下标，栈顶为下标 |
|BINARY_LSHIFT | 左移操作符（乘2)　|
|BINARY_RSHIFT | 右移操作符（除2向下取整）　|
|BINARY_AND | 按位与　|
|BINARY_XOR | 异或 |
|BINARY_OR | 按位或 |
|STORE_SUBSCR | 列表下标存储，例如Val1[Val2] = Val3 |
|DELETE_SUBSCR | 按下标删除元素，例如 del Val1[Val2] |

* 其他指令见[官方文档](https://docs.python.org/zh-cn/3.6/library/dis.html#python-bytecode-instructions)

### Python 生成的可执行程序逆向
&nbsp;&nbsp;&nbsp;&nbsp;**需要的工具有 [pyinstxtractor.py](https://github.com/extremecoders-re/pyinstxtractor)、uncompyle6、010编辑器**

1. **解压exe**,将pyinstxtractor.py放在与exe同级目录下，运行`python pyinstxtractor.py xxx.exe`,会在当前目录生成一个文件夹，里面会有一些没有后缀的文件名，xxx（exe名字）的无后缀文件就是我们需要进行反编译的文件。在正向编译过程中，生成的pyc会把python版本信息和时间戳去掉，但是反编译的时候我们需要将其补齐
2. 还存在一个没有后缀名的文件struct，用16进制编辑器打开struct和xxx文件,相同部分前面就是Python的版本号和时间戳，前面一段复制到xxx文件中，然后重命名为xxx.pyc
![0](amWiki/images/reverse/pyc/0.png "0")
![1](amWiki/images/reverse/pyc/1.png "1")
3. 接下来就使用uncompyle6转化就行了