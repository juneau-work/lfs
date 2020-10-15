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
[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/binutils-pass1.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/binutils-pass1.html)
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

### 5.5. GCC-7.2.0 - Pass 1 ###
[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/gcc-pass1.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/gcc-pass1.html)
```sh
tar -xvf gcc-7.2.0.tar.xz
cd gcc-7.2.0

tar -xvf ../mpfr-3.1.5.tar.xz
mv -v mpfr-3.1.5 mpfr
tar -xvf ../gmp-6.1.2.tar.xz
mv -v gmp-6.1.2 gmp
tar -xvf ../mpc-1.0.3.tar.gz
mv -v mpc-1.0.3 mpc

for file in gcc/config/{linux,i386/linux{,64}}.h
do
  cp -uv $file{,.orig}
  sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
	  -e 's@/usr@/tools@g' $file.orig > $file
  echo '
#undef STANDARD_STARTFILE_PREFIX_1
#undef STANDARD_STARTFILE_PREFIX_2
#define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
#define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
  touch $file.orig
done

case $(uname -m) in
  x86_64)
      sed -e '/m64=/s/lib64/lib/' \
		  -i.orig gcc/config/i386/t-linux64
;;
esac

mkdir -v build
cd       build

../configure                                       \
	--target=$LFS_TGT                              \
	--prefix=/tools                                \
	--with-glibc-version=2.11                      \
	--with-sysroot=$LFS                            \
	--with-newlib                                  \
	--without-headers                              \
	--with-local-prefix=/tools                     \
	--with-native-system-header-dir=/tools/include \
	--disable-nls                                  \
	--disable-shared                               \
	--disable-multilib                             \
	--disable-decimal-float                        \
	--disable-threads                              \
	--disable-libatomic                            \
	--disable-libgomp                              \
	--disable-libmpx                               \
	--disable-libquadmath                          \
	--disable-libssp                               \
	--disable-libvtv                               \
	--disable-libstdcxx                            \
	--enable-languages=c,c++
make && make install
```

### 5.6. Linux-4.12.7 API Headers ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/linux-headers.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/linux-headers.html)
```sh
tar -xvf linux-4.12.7.tar.xz
cd linux-4.12.7

make mrproper
make INSTALL_HDR_PATH=dest headers_install
cp -rv dest/include/* /tools/include
```

### 5.7. Glibc-2.26 ###
[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/glibc.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/glibc.html)
```sh
tar -xvf glibc-2.26.tar.xz
cd glibc-2.26

mkdir -v build
cd       build

../configure                             \
      --prefix=/tools                    \
      --host=$LFS_TGT                    \
      --build=$(../scripts/config.guess) \
      --enable-kernel=3.2             \
      --with-headers=/tools/include      \
      libc_cv_forced_unwind=yes          \
      libc_cv_c_cleanup=yes
	  
make && make install
```

### 5.8. Libstdc++-7.2.0 ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/gcc-libstdc++.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/gcc-libstdc++.html)
> Libstdc++ is the standard C++ library, Libstdc++ is part of the GCC sources.
```sh
cd $LFS/sources/gcc-7.2.0
rm -rf build
mkdir -v build
cd       build

../libstdc++-v3/configure           \
    --host=$LFS_TGT                 \
    --prefix=/tools                 \
    --disable-multilib              \
    --disable-nls                   \
    --disable-libstdcxx-threads     \
    --disable-libstdcxx-pch         \
    --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/7.2.0
make && make install

```
### 5.9. Binutils-2.29 - Pass 2 ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/binutils-pass2.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/binutils-pass2.html)
```sh
cd $LFS/sources/binutils-2.29
rm -rf build
mkdir -v build
cd       build

CC=$LFS_TGT-gcc                \
AR=$LFS_TGT-ar                 \
RANLIB=$LFS_TGT-ranlib         \
../configure                   \
    --prefix=/tools            \
    --disable-nls              \
    --disable-werror           \
    --with-lib-path=/tools/lib \
    --with-sysroot
make && make install

make -C ld clean
make -C ld LIB_PATH=/usr/lib:/lib
cp -v ld/ld-new /tools/bin

```

### 5.10. GCC-7.2.0 - Pass 2 ###

[http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/gcc-pass2.html](http://www.linuxfromscratch.org/lfs/view/8.1/chapter05/gcc-pass2.html)
```sh
rm -rf $LFS/sources/gcc-7.2.0

tar -xvf gcc-7.2.0.tar.xz
cd gcc-7.2.0
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include-fixed/limits.h
  
for file in gcc/config/{linux,i386/linux{,64}}.h
do
  cp -uv $file{,.orig}
  sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
	  -e 's@/usr@/tools@g' $file.orig > $file
  echo '
#undef STANDARD_STARTFILE_PREFIX_1
#undef STANDARD_STARTFILE_PREFIX_2
#define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
#define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
  touch $file.orig
done

case $(uname -m) in
  x86_64)
      sed -e '/m64=/s/lib64/lib/' \
		  -i.orig gcc/config/i386/t-linux64
;;
esac

tar -xvf ../mpfr-3.1.5.tar.xz
mv -v mpfr-3.1.5 mpfr
tar -xvf ../gmp-6.1.2.tar.xz
mv -v gmp-6.1.2 gmp
tar -xvf ../mpc-1.0.3.tar.gz
mv -v mpc-1.0.3 mpc

mkdir -v build
cd       build

CC=$LFS_TGT-gcc                                    \
CXX=$LFS_TGT-g++                                   \
AR=$LFS_TGT-ar                                     \
RANLIB=$LFS_TGT-ranlib                             \
../configure                                       \
    --prefix=/tools                                \
    --with-local-prefix=/tools                     \
    --with-native-system-header-dir=/tools/include \
    --enable-languages=c,c++                       \
    --disable-libstdcxx-pch                        \
    --disable-multilib                             \
    --disable-bootstrap                            \
    --disable-libgomp
make && make install
ln -sv gcc /tools/bin/cc
```
