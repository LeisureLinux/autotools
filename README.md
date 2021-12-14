#### How to generate ./configure file

```sh
    aclocal
    autoconf
    automake -a
```

or just run:

```sh
    autoreconf -ifms
```

#### INSTALLING Code

```sh
$ ./configure
$ make
$ sudo make install
```

#### UNINSTALLING

# make uninstall

# For arm-linux-gnueabihf or other similar

```sh
$  ac_cv_func_malloc_0_nonnull=yes ./configure --prefix=somewhere_you_like \
  --host=arm-linux-gnueabihf --target=arm-linux-gnueabihf
```

# For musl compile for Rasperberry Pi with static lib

```sh
export PATH=/opt/musl/arm-linux-musleabihf-cross/bin:$PATH
export T=arm-linux-musleabihf
./configure --prefix=/tmp/hello-pi-st --host=$T --target=$T ac_cv_func_malloc_0_nonnull=yes LDFLAGS=-static
make && make install
```
