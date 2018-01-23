# Linux From Scratch - Version 8.1 #

## Host System Requirements ##
[http://www.linuxfromscratch.org/lfs/view/8.1/chapter02/hostreqs.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter02/hostreqs.html)
```
centos 7.4
```
> yum -y install byacc texinfo bison patch

## Partition ##
```sh
mkfs.xfs /dev/sdc
mkdir /mnt/lfs
mount /dev/sdc /mnt/lfs
echo "export LFS=/mnt/lfs" >> ~/.bash_profile
```

-------------------------------------------------------------------------------
## Chapter03 ##

### Packages ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter03/introduction.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter03/introduction.html)
```sh
mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources
curl -s http://www.linuxfromscratch.org/lfs/view/8.1/wget-list -o
-|wget --input-file=- --continue --directory-prefix=$LFS/sources
或
curl -s http://www.linuxfromscratch.org/lfs/view/8.1/wget-list -o
-|aria2c -c -Z -i -
```

## Chapter04 ##

### 4.2 Creating the $LFS/tools Directory ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter04/creatingtoolsdir.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter04/creatingtoolsdir.html)
```sh
mkdir -v $LFS/tools
ln -sv $LFS/tools /
```

-------------------------------------------------------------------------------

### 4.3. Adding the LFS User ###

```sh
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
echo lfs:lfs|chpasswd

chown -v lfs $LFS/tools
chown -v lfs $LFS/sources

su - lfs
```

### 4.4. Setting Up the Environment ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter04/settingenvironment.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter04/settingenvironment.html)
```sh
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF

cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/tools/bin:/bin:/usr/bin
export LFS LC_ALL LFS_TGT PATH
EOF

source ~/.bash_profile
```

### 4.5. About SBUs ###

```sh
export MAKEFLAGS='-j 4'
# or
make -j4
```

-------------------------------------------------------------------------------

## Chapter05 ##
[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/generalinstructions.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/generalinstructions.html)

> **注意:**编译都以lfs身份，默认的工作目录为$LFS  
```sh
su - lfs
cd $LFS/sources
```
### 5.4. Binutils-2.29 ###
```sh
tar -xvf binutils-2.29.tar.bz2
cd binutils-2.29

mkdir -v build
cd       build

../configure --prefix=/tools            \
             --with-sysroot=$LFS        \
			 --with-lib-path=/tools/lib \
			 --target=$LFS_TGT          \
			 --disable-nls              \
			 --disable-werror

make

case $(uname -m) in
	x86_64) mkdir -v /tools/lib && ln -sv lib /tools/lib64 ;;
esac

make install
```

### 5.5. GCC-7.2.0 ###
```sh

```

