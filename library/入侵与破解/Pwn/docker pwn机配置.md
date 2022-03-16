# docker pwn机配置

## 使用方法

```
pwn机密码pwn114514
```

```
docker pull woodwhale/vanpwn:latest
```

```
docker run --name vanpwn --hostname vanpwn -p 15901:5901 -p 16901:6901 --shm-size=1g --user=woodwhale -it woodwhale/vanpwn:latest /home/woodwhale/.dockerinit
```

多出了如下功能（自行探索噢）
![1](amWiki/images/pwn/1.png "1")

## 前言
由于自己的荣耀轻薄本装了双系统（ubuntu20.04和win10），每次在打ctf比赛的时候基本上都是使用ubuntu系统来做pwn和misc，每次做学校的作业基本都要用office之类的工具（ubuntu虽然也有wps支持但是还是挺不习惯的）。加上近期学了docker的部分知识，于是先要用docker+ubuntu+vnc+pwntools的方式配置一台属于自己的docker pwn机。
虽然github上已经有很多版本的pwn机，但是还是想自己动手做一台。
docker中ubuntu容器用的xubuntu-desktop最小安装，加上vnc服务其实看起来还行，本来是打算安装gnome-desktop或者kde的，但是太大并且下载太慢，就没有选择了
docker容器的vnc配置可以参考视频，还有配套的博文
至于配置好了vnc服务之后，还需要配置一系列的pwntools，我们就从这一部分开始了！
我这里直接启动了我之前做好的镜像(/home/woodwhale/.dockerinit是我自己写的一个sh脚本，目的是启动直接开启vnc服务)
docker run --name pwn --hostname pwn -p 15901:5901 --shm-size=1g --user=woodwhale -itd bi0xpwn:v2 /home/woodwhale/.dockerinit
![2](amWiki/images/pwn/2.png "2")
启动后连接vnc的效果：
![3](amWiki/images/pwn/3.png "3")
比虚拟机快了n倍，很爽！
## 1、docker解封
这一部分我姑且叫他解封，因为docker的容器是精简版本的，所以阉割了很多库和功能，我们首先需要进入容器使用
unminimize
我这里之前已经执行过了，所以没有反映，建议在配置vnc服务之前就输入这个指令，之后输入需要比较久的时间！
![4](amWiki/images/pwn/4.png "4")

##　2、切换apt源
机房10m的网，如果用ubuntu中国的官方源我只有28kb左右的速度，用了清华的瞬间2mb的速度，虽然在安装32位库的方面非官方源可能出点问题，但是先用网速快的再说！
```
sudo vim /etc/apt/source.list
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
sudo apt update
```

## 3、安装需要的pwntools！
### 1.git
必须经常在github上下载工具，所以git必装
sudo apt install git
![5](amWiki/images/pwn/5.png "5")
安装完后，初始化一下
git init
git submodule update
### 2.gcc
编程语言译器，做写c必须的，不解释了
sudo apt install gcc
安装成功后验证：
![6](amWiki/images/pwn/6.png "6")

### 3.python3-pip
虽然docker在unminimize之后安装了python3，但是没有pip3
sudo apt install python3-pip
![7](amWiki/images/pwn/7.png "7")

### 4.python2
有些库还是只有python2支持的，所以还是得下一个python2
（懂不懂py2的含金量啊~）
sudo apt install python2
![8](amWiki/images/pwn/8.png "8")

### 5.curl和wget
我一般用来下载东西的，其实也可以做做web题，post、get请求都是可以的
sudo apt install curl
sudo apt install wget
### 6.pip2
pip2已经从apt中删除了，通过curl的方式下载一个py文件进行安装
需要用root权限安装pip2，不然有些东西安装不了会失败
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
sudo python2 get-pip.py
![9](amWiki/images/pwn/9.png "9")
![10](amWiki/images/pwn/10.png "10")

验证pip2
![11](amWiki/images/pwn/11.png "11")

### 7.pwntools
pwntools是一个CTF框架和开发库。它是用Python编写的，设计用于快速原型和开发，旨在使开发编写尽可能简单
直接使用pip3和pip2安装python3和python2的版本~
pip3 install pwntools
pip2 install pwntools
![12](amWiki/images/pwn/12.png "12")

### 8.pwndbg
我个人做题都是用pwndbg的，所以这里配置pwndbg的环境
首先去git上克隆一下
git clone https://github.com/pwndbg/pwndbg.git
我这里机房的网给墙了，所以直接用win主机在官网下载zip包了
![13](amWiki/images/pwn/13.png "13")

然后使用docker的cp指令直接存到容器中
 docker cp .\pwndbg-dev.zip pwn:/home/woodwhale/workspace/ctftools/pwntools
![14](amWiki/images/pwn/14.png "14")

到容器中使用unzip指令解压
woodwhale@pwn:~/workspace/ctftools/pwntools$ unzip pwndbg-dev.zip
然后到文件夹中，输入如下开始安装必要环境
chmod +x ./setup.sh
./setup.sh
![15](amWiki/images/pwn/15.png "15")

等待一会就安装好啦！
![16](amWiki/images/pwn/16.png "16")

验证安装成功
![17](amWiki/images/pwn/17.png "17")

### 9.pwngdb
pwngdb可以配合pwndbg使用，增加了很多方法来查看，可以参考这篇博文进行配置
首先克隆下来，由于网络问题我还是在win宿主机上下载了zip包，通过docker cp指令移动到容器中
git clone https://github.com/scwuaptx/Pwngdb.git 
docker cp .\pwngdb.zip pwn:/home/woodwhale/workspace/ctftools/pwntools
![18](amWiki/images/pwn/18.png "18")
仍然是unzip
![19](amWiki/images/pwn/19.png "19")

然后就是配置主目录下的.gdbinit文件
我的配置如下：
```
source /home/woodwhale/workspace/ctftools/pwntools/pwndbg/gdbinit.py
source /home/woodwhale/workspace/ctftools/pwntools/Pwngdb/pwngdb.py
source /home/woodwhale/workspace/ctftools/pwntools/Pwngdb/angelheap/gdbinit.py

define hook-run
python
import angelheap
angelheap.init_angelheap()
end
end
```

三个source需要正确的位置
### 10.one_gadget
一键获取shell的one_gadget工具！
首先需要安装ruby这门语言，在其包管理工具gem中下载安装one_gadget
sudo apt install ruby ruby-dev
sudo gem install one_gadget
验证安装
![20](amWiki/images/pwn/20.png "20")

### 11.seccomp-tools
seccomp-tools用来看沙箱题，也是通过gem来安装！
sudo gem install seccomp-tools
验证安装成功
![21](amWiki/images/pwn/21.png "21")

### 12.LibcSearcher
老版本的 libcsearcher就不说了，这里用的是新版本通过libcdatabase在线查询的libcsearcher
[github项目地址](https://github.com/dev2ero/LibcSearcher)
安装也很简单，使用pip3就可以简单一键安装！
pip3 install LibcSearcher
![22](amWiki/images/pwn/22.png "22")

### 13.patchelf
更改libc的神器！
安装也很简单！
sudo apt install patchelf 
验证安装：
![23](amWiki/images/pwn/23.png "23")

### 14.glibc-all-in-one
配合patchelf使用，基本都能用！
github[项目地址](https://github.com/matrix1001/glibc-all-in-one)
由于网路问题，我是用win宿主机下载zip文件docker cp进入容器
![24](amWiki/images/pwn/24.png "24")
进入容器unzip一下
![25](amWiki/images/pwn/25.png "25")
将其中的updata_list文件的首行改为python3，因为我这里无法识别python这个指令（只有python3和python2）
![26](amWiki/images/pwn/26.png "26")
然后在当前文件夹下打开终端，给所有可执行文件赋予权限后下载libc全家桶
chmod +x *
./update_list
然后就是一顿download
![27](amWiki/images/pwn/27.png "27")
配合我自己写的一个libcpatcher.py脚本，可以进行libc选择！
```
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import sys, os, subprocess

LIBS_PATH = "/home/bi0x/ctftools/pwntools/glibc-all-in-one/libs/" # 使用自己glibc/libs的目录
BINARY = ""
LIBC = ""
BIT = ""

def check_args():
    if not (3 >= len(sys.argv) >= 2) :
        print("\033[31mParameter error!Try again!\033[0m")
        print("\033[35mUsage   :  libcpatcher elfname (libc_addr)\033[0m")
        print("\033[35mExample :  libcpatcher hacknote 2.23-0ubuntu3_amd64\033[0m")
        exit(0)

def choose_libc():
    print("\033[32mYou can choose: \033[0m")
    libs_list = os.listdir(LIBS_PATH)
    libs_list.sort()
    for i in range(0,len(libs_list)):
        print(f"\033[31m{i+1}\033[0m \033[33m:\033[0m \033[36m{libs_list[i]}\033[0m")
    i = int(input("\033[32mChoose: \033[0m"),10) - 1
    if i >= len(libs_list) :
        print("\033[31mChoose error!Try again!\033[0m")
        exit(0)
    return libs_list[i]

def init():
    global BINARY, LIBC, BIT
    BINARY = sys.argv[1]
    LIBC = sys.argv[2] if len(sys.argv) == 3 else choose_libc()
    BIT = subprocess.check_output([f"file {BINARY}"],shell=True).decode().split(" ")[2]

def libcpatcher():
    global BINARY, LIBC, BIT
    error = "\033[31mText file busy!\033[0m"
    try:
        print("\n\033[31mBefor patch \033[0m")
        print(subprocess.check_output([f"ldd {BINARY}"],shell=True).decode())
        if "64" in BIT and "amd64" in LIBC:
            subprocess.check_output([f"patchelf --set-rpath {LIBS_PATH}{LIBC}/:/libc.so.6 {BINARY}"],shell=True)
            subprocess.check_output([f"patchelf --set-interpreter {LIBS_PATH}{LIBC}/ld-linux-x86-64.so.2 {BINARY}"],shell=True)
        elif "32" in BIT and "i386" in LIBC :
            subprocess.check_output([f"patchelf --set-rpath {LIBS_PATH}{LIBC}/:/libc.so.6 {BINARY}"],shell=True)
            subprocess.check_output([f"patchelf --set-interpreter {LIBS_PATH}{LIBC}/ld-linux.so.2 {BINARY}"],shell=True)
        else:
            error = "\033[31mlibc Bit error!Try again!\033[0m"
            exit(0)
        print("\033[32m======================================================================\033[0m \n")
        print("\033[31mAfter patch \033[0m")
        print(subprocess.check_output([f"ldd {BINARY}"],shell=True).decode())
    except:
        print(error)
        exit(0)
    
def main():
    check_args()
    init()
    libcpatcher()
    
if __name__ == '__main__':
    main()
```
### 15.补充
python3和python2的pwntools库需要pathlib2的支持，所以我们还得安装一下
pip3 install pathlib2
pip2 install pathlib2
另外，单纯的安装pwntools之后可能出现工具无法使用的情况，例如无法使用checksec和ROPgadget，我们可以在.bashrc文件中导入~/.local/bin的path，因为这个文件夹中都是我们的pwntools的小工具，导入到环境变量中就可以使用啦
![28](amWiki/images/pwn/28.png "28")
在~/.bashrc最后加入这句话
export PATH=$PATH:/home/woodwhale/.local/bin
![29](amWiki/images/pwn/29.png "29")
## 4、docker commit
最后当然是将我们的配置好的docker pwn打包成新的镜像啦
docker commit <id/name> 仓库：名字
![30](amWiki/images/pwn/30.png "30")
## 5、vnc的补充
如果觉得每次都需要开启vncviewer来连接麻烦，可以使用novnc来进行web端的远程连接
首先
sudo apt install novnc
然后
websockify -D --web=/usr/share/novnc/  6901 localhost:5901
我把这两句运行指令直接放在了.dockerinit中，容器运行就启动
![31](amWiki/images/pwn/31.png "31")
## 后话
配置了这么久的docker pwn终于是结束了，已经上传到dockerhub上了，自己pull一下就可以用了
```
docker pull woodwhale/vanpwn:latest
```
```
docker run --name vanpwn --hostname vanpwn -p 15901:5901 -p 16901:6901 --shm-size=1g --user=woodwhale -it woodwhale/vanpwn:latest /home/woodwhale/.dockerinit
```