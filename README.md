# tensorflow_on_farber

#### GCC
--------------
Download the GCC 4.9.4 from source, compile, install.
_this is because of `as` error in bazel_

```
#!/bin/sh

unset LIBRARY_PATH CPATH C_INCLUDE_PATH 
unset PKG_CONFIG_PATH CPLUS_INCLUDE_PATH INCLUDE LD_LIBRARY_PATH

cd gcc-4.9.4
./contrib/download_prerequisites

mkdir objdir
cd objdir


# I've added --disable-multilib to fix the following error:
# /usr/bin/ld: crt1.o: No such file: No such file or directory
# collect2: ld returned 1 exit status
# configure: error: I suspect your system does not have 32-bit 
# developement libraries (libc and headers). If you have them,
# rerun configure with --enable-multilib. If you do not have them, 
# and want to build a 64-bit-only compiler, rerun configure 
# with --disable-multilib.           

../configure --prefix=$HOME/opt/gcc-4.9.4 \
             --disable-multilib \
             --disable-nls \
             --enable-languages=c,c++ \
             --with-ld=/usr/bin/ld \
             --with-nm=/usr/bin/nm \
             --with-as=/usr/bin/as

make        
make install
```

Create a separate `use_gcc_4.9.4.sh` file, with necessary environment variables.
```
#!/bin/sh
this=$HOME/opt/gcc-4.9.4
export PATH=$this/bin:$PATH
export CPATH=$this/include:$CPATH
export LIBRARY_PATH=$this/lib:$LIBRARY_PATH
export LIBRARY_PATH=$this/lib64:$LIBRARY_PATH
export LD_LIBRARY_PATH=$this/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$this/lib64:$LD_LIBRARY_PATH
```

use `source use_gcc_4.9.4.sh` to change the compile environment 

1. ```
 git clone https://github.com/tensorflow/tensorflow
 cd tensorflow
 git checkout r1.0
 ```
 
2. 
