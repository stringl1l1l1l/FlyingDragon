# FlyingDragon
从零开始的移植之路：软件杯——thunderbird78移植到龙芯平台

开发环境:
Loongnix桌面操作系统
**[thunderbird官方文档](https://developer.thunderbird.net/thunderbird-development/getting-started)**
**[markdown语法教程](https://markdown.com.cn/basic-syntax/headings.html)**
- [FlyingDragon](#flyingdragon)
  - [环境配置](#环境配置)
    - [error1](#error1)
    - [solve1](#solve1)
    - [error2](#error2)
    - [solve2](#solve2)
    - [error3](#error3)
  - [移植之路](#移植之路)
    - [solve3](#solve3)
## 环境配置
下载龙芯已移植的旧版本thunderbird依赖项
```
sudo apt build-dep thunderbird
```
拉取thunderbird78源代码
```
git clone --depth=1 --branch=debian/1%78.14.0-1_deb11u1 https://salsa.debian.org/mozilla-team/thunderbird.git</code>
```
拉取成功后，进入目录，在mozconfig中添加编译thunderbird的配置
```
cd ./thunderbird
echo 'ac_add_options --enable-project=comm/mail' > mozconfig
```
可选以debug模式编译，但编译速度会变慢，暂时不加
```
echo 'ac_add_options --enable-debug' >> mozconfig
```
至此，尝试build项目
```
./mach build
```

### error1
于是遇到了迁移过程中的第一个报错:
```
......
ERROR: Command `sh /home/loongson/SoftWareCup/thunderbird/build/moz.configure/../autoconf/config.guess` failed with exit status 1.
```
### solve1
这个错误出现的原因是config.guess脚本版本太旧，无法猜测龙芯架构的系统，因此使用下面的代码更新:
```
cd ./build/autoconf
wget -O ./config.sub "git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD"
wget -O ./config.guess "git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD"
```

### error2
继续build，遇到下面的错误:
```
AttributeError: 'ValueError' object has no attribute 'message'
  ......
  File "/home/loongson/SoftWareCup/thunderbird/build/moz.configure/init.configure", line 844, in real_host
    die(e.message)
```
### solve2
这个问题出现的原因是，对于python的较高版本，异常类不再有message属性，根据提示将`init.configure`脚本修改为：
```
841     try:
842         return split_triplet(host)
843     except ValueError as e:
844         die(str(e))
```

### error3
继续build，遇到下面的错误：
```
DEBUG: Executing: `sh /home/loongson/SoftWareCup/thunderbird/build/moz.configure/../autoconf/config.guess`
DEBUG: Executing: `sh /home/loongson/SoftWareCup/thunderbird/build/moz.configure/../autoconf/config.sub loongarch64-unknown-linux-gnu`
ERROR: Unknown CPU type: loongarch64
```
可以看到是thunderbird代码中没有对loongarch64架构的支持，至此环境配置基本差不多了，算是正式开始了thunderbird移植之路。


## 移植之路
### solve3
