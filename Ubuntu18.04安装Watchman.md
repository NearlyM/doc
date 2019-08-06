# Ubuntu18.04安装Watchman

ubuntu安装watchman，过程中可能会遇到一些问题，现在记录如下：

```
git clone https://github.com/facebook/watchman.git //步骤1
cd watchman/
git checkout v4.9.0  //步骤2
sudo apt-get install -y autoconf automake build-essential python-dev libssl-dev libtool		//步骤3
./autogen.sh
./configure
make
sudo make install
```

步骤1：从github上下载或者clone项目watchman

步骤2：切换分支到v4.9.0(使用时最新的稳定版本)，注意v4.9.0是tag版本，直接使用

步骤3：执行完提示pgk-config没有安装，则需要安装`sudo apt-get install pgk-config`

其余按上面步骤执行，安装结束就可以使用watchman了