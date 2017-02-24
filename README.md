# tensorflow_on_farber

####GCC
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

####cudnn
------------

Download the cudnn 5.1 and extraced to `~/usr/cuda/7.5v5.1` as `$cudnn_root`

####login_rc
-----------------
```
source ~/use_gcc_4.9.4.sh
vpkg_require cuda/7.5
PATH=~/usr/cuda/7.5v5.1/include:$PATH
export LD_LIBRARY_PATH=~/usr/cuda/7.5v5.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/opt/shared/cuda/7.5.18/lib64:/opt/shared/cuda/7.5.18/extras/CUPTI/lib64"
export CUDA_HOME=/opt/shared/cuda/7.5.18
vpkg_require anaconda/2.5.0:python2
source activate ~/conda_env
```


####Bazel
------------

1. `git clone https://github.com/bazelbuild/bazel.git && cd bazel`

2. Edit `tools/cpp/CROSSTOOL`
  * Replace all occurrences of `/usr/bin/gcc` with `gcc path`
  * Replace all occurrences of `/usr/bin/cpp` with `cpp path`
  * After the toolpath containing `gcc path`, add the lines
    * `linker_flag: "-Wl,-Rlib64 path"`
    * `cxx_builtin_include_directory: "include1 dir"`
    * `cxx_builtin_include_directory: "include2 dir"`
    * `cxx_builtin_include_directory: "include3 dir"`

3. Edit `scripts/bootstrap/buildenv.sh`
  * Comment out `atexit "rm -fr ${DIR}"`
  
4. `export EXTRA_BAZEL_ARGS='-s --verbose_failures --ignore_unsupported_sandboxing --genrule_strategy=standalone --spawn_strategy=standalone --jobs 8'`

5. `./compile.sh`


####TensorFlow
----------------

1. use `source use_gcc_4.9.4.sh` to change the compile environment 

2. Download tensorflow from source 
 ```
 git clone https://github.com/tensorflow/tensorflow
 cd tensorflow
 git checkout r1.0
 ```

3. Edit `third_party/gpus/crosstool/CROSSTOOL`, making the same changes we made for Bazel. (/usr/bin/gcc etc. likely won't need to be replaced, though.)

4. Edit `third_party/gpus/crosstool/clang/bin/crosstool_wrapper_driver_is_not_gcc` Replace all `/usr/bin/gcc` with `gcc path`.

5. If you're not using default system gcc (e.g. if you had to compile newer gcc, like discussed above), add the following linker flags to `tensorflow/third_party/gpus/crosstool/CROSSTOOL.tpl`, line 59:
  * `linker_flag: "-L/home/username/localinst/opt/gcc-4.9.4/lib64"`
  * `linker_flag: "-Wl,-rpath,/home/username/localinst/opt/gcc-4.9.4/lib64"`
  
6. `./configure`
 * disable `jemalloc`
 * enable `cuda` and `cudnn`
 * others with default (press `Enter`)
