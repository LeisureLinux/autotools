#### How to generate ./configure file

    ```sh
    aclocal
    autoconf
    automake -a
    ```

    Or just run:

    ```sh
        autoreconf -ifms
    ```

#### Install source code natively

    ```sh
        $ ./configure
        $ make
        $ sudo make install
    ```

#### 卸载

    > make uninstall

#### 树莓派的交叉编译（方法一）Cross Compile for Rasperberry Pi (Option 1)

- For arm-linux-gnueabihf or other similar

  ```sh
  $  ac_cv_func_malloc_0_nonnull=yes ./configure --prefix=somewhere_you_like
    --host=arm-linux-gnueabihf --target=arm-linux-gnueabihf
  ```

- For musl compile for Rasperberry Pi with static lib
  ```sh
  export PATH=/opt/musl/arm-linux-musleabihf-cross/bin:$PATH
  export T=arm-linux-musleabihf
  ./configure --prefix=/tmp/hello-pi-st --host=$T --target=$T ac_cv_func_malloc_0_nonnull=yes LDFLAGS=-static
  make && make install
  ```

#### 创建 Debian 包(How to create Debian package)

1. GPG Key
   ```sh
     MYNAME="Jame Smith"
     gpg --gen-key
     gpg -a --output ~/.gnupg/$MYNAME.gpg --export "$MYNAME"
     gpg --import ~/.gnupg/$MYNAME.gpg
   ```
2. Install required packages
   ```sh
   sudo apt-get install build-essential autoconf automake \
   autotools-dev dh-make debhelper devscripts fakeroot \
   xutils lintian pbuilder
   ```
3. Prepare Source Code
   ```sh
   make dist
   mkdir PKG_DIR && cd PKG_DIR
   mv ../helloworld-1.1.2.tar.gz . && tar xf helloworld-1.1.2.tar.gz
   cd helloworld-1.1.2
   ```
4. Debianization
   ```sh
   dh_make -e youremail@address -f ../helloworld-1.1.2.tar.gz
   ```
   选择 single binary
5. 检查依赖包
   ```sh
   dpkg-depcheck -d ./configure
   ```
6. 修改 debian/control 文件, 为了树莓派打包，compatbility 版本要修改为 10
   - Section
   - Build-Depends: debhelper (>= 10~), autotools-dev
   - Homepage
   - Description
   - Architecutre ?
   - echo 10 > debian/compat
   - [DH_COMPAT 版本详细文档](https://manpages.debian.org/testing/debhelper/debhelper.7.en.html)
7. 修改其他文件，例如 copyright, changelog
8. 正式打包，（-us/-uc=不签名）

   ```sh
   dpkg-buildpackage -rfakeroot -kyouremail@address
   # 验证打包
   lintian ../helloworld_0.1-1_amd64.deb
   ```

#### 使用 pbuilder 打包成树莓派 raspbian OS 的安装包[参考文章]（https://grid.in.th/2020/10/cross-compile-packages-for-raspbian-from-x86_64/）

1. 安装相关的依赖包
   ```sh
    sudo apt install pbuilder qemu-user-static curl ubuntu-keyring debian-archive-keyring
   ```
2. 生成 /root/.pbuilderrc 文件如下：

   ```sh
   #!/bin/bash
   set -e

   if [ "$OS" == "debian" ]; then
   MIRRORSITE="http://ftp.jp.debian.org/debian/"
   COMPONENTS="main contrib non-free"
   DEBOOTSTRAPOPTS=("${DEBOOTSTRAPOPTS[@]}" "--keyring=/usr/share/keyrings/debian-archive-keyring.gpg")
       : ${DIST:="buster"}
       : ${ARCH:="amd64"}
   elif [ "$OS" == "raspbian" ]; then
       # MIRRORSITE="https://archive.raspbian.org/raspbian/"
       MIRRORSITE="http://mirrors.sjtug.sjtu.edu.cn/raspbian/raspbian/"
       COMPONENTS="main contrib non-free rpi"
       DEBOOTSTRAPOPTS=("${DEBOOTSTRAPOPTS[@]}" "--keyring=/usr/share/keyrings/raspbian-archive-keyring.gpg")
           PBUILDERSATISFYDEPENDSCMD="/usr/lib/pbuilder/pbuilder-satisfydepends-apt"
           : ${DIST:="buster"}
           : ${ARCH:="armhf"}
   elif [ "$OS" == "ubuntu" ]; then
       MIRRORSITE="http://jp.archive.ubuntu.com/ubuntu/"
       COMPONENTS="main restricted universe multiverse"
       DEBOOTSTRAPOPTS=("${DEBOOTSTRAPOPTS[@]}" "--keyring=/usr/share/keyrings/ubuntu-archive-keyring.gpg")
   else
       echo "Unknown OS: $OS"
       exit 1
   fi

   if [ "$DIST" == "" ]; then
       echo "DIST is not set"
       exit 1
   fi

   if [ "$ARCH" == "" ]; then
       echo "ARCH is not set"
       exit 1
   fi

   NAME="$OS-$DIST-$ARCH"

   if [ "$ARCH" == "armel" ] && [ "$(dpkg --print-architecture)" != "armel" ]; then
       DEBOOTSTRAP="qemu-debootstrap"
   fi
   if [ "$ARCH" == "armhf" ] && [ "$(dpkg --print-architecture)" != "armhf" ]; then
       DEBOOTSTRAP="qemu-debootstrap"
   fi

   DEBOOTSTRAPOPTS=("${DEBOOTSTRAPOPTS[@]}" "--arch=$ARCH")
   BASETGZ="/var/cache/pbuilder/$NAME-base.tgz"
   DISTRIBUTION="$DIST"
   BUILDRESULT="/var/cache/pbuilder/$NAME/result/"
   APTCACHE="/var/cache/pbuilder/$NAME/aptcache/"
   BUILDPLACE="/var/cache/pbuilder/build"
   HOOKDIR="/var/cache/pbuilder/hook.d/"
   ```

   3. 创建树莓派 Image，并登录进入 Image，下载安装 curl，并安装仓库源

   ```sh
   sudo mkdir -p /var/cache/pbuilder/raspbian-stretch-armhf/aptcache/
   # 因为设置镜像到上海交大了，所以这步 create 的进度不算太慢
   sudo OS=raspbian ARCH=armhf DIST=stretch pbuilder --create --distribution stretch
   sudo OS=raspbian DIST=stretch ARCH=armhf pbuilder login --save-after-login
   # 更新仓库源
   apt update
   apt install curl
   curl http://archive.raspberrypi.org/debian/raspberrypi.gpg.key | apt-key add -
   cat <<EOF > /etc/apt/sources.list.d/raspi.list
       deb http://archive.raspberrypi.org/debian/ stretch main
   EOF
   apt update
   # 校验
   apt cache policy
   # 退出树莓派 Image
   exit
   ```

   4. 根据 dsc 包描述文件，创建树莓派的软件包

   ```sh
   sudo OS=raspbian DIST=stretch ARCH=armhf pbuilder --build ./helloworld_1.1.2-1.dsc
   ```
