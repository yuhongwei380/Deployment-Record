## GCC install 

1.download GCC source pacakacges

```
https://ftp.gnu.org/gnu/gcc/
```

select the target version you need,now we download GCC 9.3.0

```
wget https://ftp.gnu.org/gnu/gcc/gcc-9.3.0/gcc-9.3.0.tar.gz
```

2.download dependencies

```
./contrib/download_prerequisites   
```

if you meet failed about gmp-6.1.0

```
./contrib/download_prerequisites --no-verify
```

3.create a precompiled directory

```
mkdir build && cd build
```

4.set compile options and compile

```
../configure --prefix=/usr/local/gcc-9.3.0 --enable-bootstrap --enable-checking=release --enable-languages=c,c++ --disable-multilib
```

–-enable-languages表示你要让你的gcc支持哪些编程语言;
-–disable-multilib表示编译器不编译成其他平台的可执行代码;
-–disable-checking表示生成的编译器在编译过程中不做额外检查
–-enable-checking=xxx 表示编译过程中增加XXX检查
–prefix=/usr/local/gcc-9.3.0 指定安装路径
–enable-bootstrap 表示用第一次编译生成的程序进行第二次编译，然后用再次生成的程序进行第三次编译，并且检查比较第二次和第三次结果的正确性，也就是进行冗余的编译检查工作。 非交叉编译环境下，默认已经将该值设为 enable，可以不用显示指定；交叉编译环境下，需要显示将其值设为 disable。

5.make

```
make 
make install
```

