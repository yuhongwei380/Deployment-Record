# Python3 

1.install 

centos7

```
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

ubuntu：

```
sudo apt-get install build-essential zlib1g-dev libbz2-1.0 libssl-dev libncurses5-dev libsqlite3-dev libreadline-dev tk-dev libgdbm-dev libdb5.3 libpcap-dev xz-utils libexpat1-dev liblzma-dev libssl-dev openssl libffi-dev libc6-dev
```

2.download

```
https://www.python.org/downloads/release/
```

3.create  python3

```
mkdir /usr/local/python3
```

4.move packages

```
mv Python-3.7.0  /usr/local/python3
```

5.make&& make install

```
cd /usr/local/python3/Python-3.7.0  
./configure  --enable-optimizations --prefix=/usr/local/python3
make -j8 && make altinstall
```

6.create soft link

```
ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip3
```

