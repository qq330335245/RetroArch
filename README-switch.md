
## 环境搭建

官方参考链接（真的只能作为参考，截止文档编写前（2022/6/8）此文档的最近更新时间为2020年，跟着文档走有很多地方都走不通，具体一些细节在下面）：
https://docs.libretro.com/development/retroarch/compilation/switch-libnx/
细节1：
   文档里给的依赖包安装指令dkp-pacman -Sy devkit-env devkitA64 libnx switch-tools switch-mesa switch-zlib switch-bzip2 switch-freetype switch-libpng
   在windows下要把dkp-pacman改成pacman
   删掉devkit-env，这个包貌似不再支持了，会找不到这个包而报错
细节2：
   国内用户由于特殊原因需要科学绿色通道（你懂的），如果在安装依赖包时遇到网络的问题，在绿色通道的前提下还是没办法解决的话，考虑在host文件里加上ip域名映射，具体ip地址自行去dns网查
细节3：
   在部署环境时可能会出现找不到几个库函数，大致是not found:--------------lXXXXXXXX,
   解决办法是：把l后面的XXXXXXXX复制下来，去谷歌搜索，一般会在前几个条目显示出对应的函数的包名，把这个包名记住，在msys2控制台下输入pacman -Sl，会显示出当前所有可以安装的包名，全选复制粘贴到一个文本文档里，然后在这里查找刚才在谷歌找到的包名，找到一个 switch-包名 的匹配后，复制switch-包名，然后回到msys2输入 pacman -Sy switch-包名
   例子：报了not found:--------------lswresample，复制swresample到谷歌，谷歌给出相关的关键字是FFmpeg，然后回到列出的所有包名里查找它，会找到switch-ffmpeg，然后在msys2输入pacman -Sy switch-ffmpeg
   如果找不到对应的包名，可能是谷歌得到的关键字有误，继续找

## 编译和构建

经研究后发现，switch版的retroarch有个特点（缺点）：每个核心都是<核心+前端>的形式，具体原因不清楚，猜测是switch系统的限制吧。
这个形式会导致一个很坑的问题，就是如果我要修改前端的功能，理论上我也要把对应的核心都要重新构建一遍，而构建核心花费的时间真的很长。。。

### 编译核心

在msys2下cd到libretro-super下输入：
./libretro-fetch.sh 核心1 核心2 核心3
platform=libnx ./libretro-build.sh 核心1 核心2 核心3

参数长度可选，想编译一个就直接填入一个核心的名字就行了，具体支持的核心以及对应的名称可以到libretro-super项目下的recipes/nintendo/libnx查看

### 构建核心

正如我前面所说，switch版的核心都是<核心+前端>的，编译出来的核心还只是XXX.a的中间件，还需要在retroarch项目下构建才能变成.nro的真正核心文件
把编译出来后的XXX.a文件复制到retroarch目录下，命名为libretro_libnx.a，然后在msys2下cd到libretro-super/retroarch下输入：
make -f Makefile.libnx
就可以在同样目录下得到retroarch_switch.nro的核心文件（其实就是带核心的retroarch前端...）

### 构建retroarch前端

又如我前面所说，其实我们构建出来的retroarch_switch.nro就是完整的retroarch了，只不过是带唯一一个核心的，那如果我硬是要一个啥核心都不带的retroarch前端呢？
很遗憾，经过一番摸索，并没有找到官方提供的单独构建无核心前端，但是自己多次尝试之后，终于成功单独构建出来了，细节是修改了retroarch/Makefile.libnx,
然后新建了一个retroarch/Makefile.libnx_no_core,构建的方式差不多，在retroarch目录下输入：
make -f Makefile.libnx_no_core
就ok了

## 杂项
其实研究了下发现官方是有做了一个自动构建的脚本的，在libretro-super/libretro-buildbot-recipe.sh
只需要执行：
./libretro-buildbot-recipe.sh recipes/nintendo/libnx
就可以完整自动构建所有核心和前端还有资源文件，其实就相当于发布吧，但是这个流程是真的很长，
如果只想要自动构建几个核心，也可以把recipes/nintendo/libnx里的部分需要构建的核心留下，其他删掉，那么自动发布的时候就会只构建这些而减少了很多时间