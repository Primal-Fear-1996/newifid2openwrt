# newifid2openwrt
自编译路由器OPENWRT固件 请上传.config文件 actions内选择build openwrt -- run workflow

windows下使用wsl编译
1.安装WSL

windows系统启用WSL功能(Home edition无法启用)

windows10（版本号至少1903）或windows11系统，以管理员身份运行powershell，输入

dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

安装wsl

wsl --install -d Ubuntu

按WIN键+R，运行wsl.exe,启动wsl，设置用户名与密码,编译时不要使用root用户

为wsl配置国内apt源

sudo echo > /etc/apt/sources.list

sudo cat << EOF > /etc/apt/sources.list
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

EOF

2.准备环境

sudo apt update && sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync

从下一步必须开始确保网络访问github.com等网站无障碍

配置wsl系统走http代理

export ALL_PROXY=http://127.0.0.1:3128 && source ~/.bashrc

其中127.0.0.1与3128端口换成你自己实际使用的

设置好go模块代理

export GOPROXY=https://goproxy.io,direct && source ~/.bashrc

cd ~/进入使用者用户目录

使用 git clone https://github.com/coolsnowwolf/lede 命令下载好源代码，然后 cd lede 进入目录

添加一些必须的软件源码

sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default && sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default

./scripts/feeds update -a && ./scripts/feeds install -a

3.首次编译

make menuconfig

此时图形界面选择编译选项，选择需要的应用，键盘上下移动光标，回车进入菜单，esc返回，y选择，n去除选择。如果刷机到路由器上注意固件大小限制

(1)启用ipv6需要在Extra packages中选择ipv6helper

(2)启用samba4

Extra packages里取消勾选autosamba.

LuCI – Applications里取消勾选luci-app-samba，勾选luci-app-samba4.

Network里取消samba36-server.

(3) 如果需要cloudflare ddns服务

Network–IP Addresses and Names里勾选ddns-scripts_cloudflare.com-v4

SAVE保存.config文件，连按esc退出编译选项界面

输入source /etc/environment，这一步很重要，不输这段代码肯定会报错。

make -j8 download V=s 下载dl库

make -j1 V=s（-j1 后面是线程数。第一次编译推荐用单线程）即可开始编译你要的固件

编译完毕后，cd ~/lede/bin/targets ，该目录存放编译好的固件，将sysupgrade 固件用breed写入路由器，如果路由器固件大小超过限制，将没有sysupgrade固件
