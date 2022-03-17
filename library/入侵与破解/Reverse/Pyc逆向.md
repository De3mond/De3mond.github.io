# Pyc逆向和Python的可执行程序（exe）逆向

## Pyc逆向
1. 此类题目比较简单，只需要下载一个插件叫 uncompyle6 `pip install uncompyle6`就可以了
2. 使用方法是`uncompyle6 -o test.py test.pyc`,将 pyc 文件反编译写进 py 文件中
3. 再查看 Python 程序中的内容解出 flag ，通常此类题目都是结合比较复杂的密码题出的