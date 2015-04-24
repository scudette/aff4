---
layout: docs
title: Building a statically linked imager.
order: 1
---

The following instructions are for building a statically linked imager for Linux
and Windows. Unfortunately, Linux distributions no longer ship statically
compiled libraries, hence one needs to build all dependencies themselves.

In the following we create a directory structure for static libraries set using
--prefix. This will force dependencies to be built from there instead of using
the system's libraries.

## Building for Linux.

```
export PREFIX=/home/scudette/build/static/
export CXXFLAGS="-I$PREFIX/include"

apt-get source libraptor2-dev
cd raptor2-2.0.13/
./configure --prefix=$PREFIX --enable-static --without-www LDFLAGS="-L/home/scudette/build/static/lib -static -static-libstdc++" --enable-serializers="turtle ntriples" --enable-parsers="turtle ntriples"
make -j4 install
cd ..

apt-get source libpcre++-dev
cd libpcre++-0.9.5/
./autogen.sh
cp /usr/bin/libtool ./libtool
./configure --prefix=$PREFIX  --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++"
make -j4 install
cd ..

apt-get source gflags
cd gflags-2.0/
./configure --prefix=$PREFIX  --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++"
make -j4 install
d ..

apt-get source  libgoogle-glog-dev
cd google-glog-0.3.3/
./configure --prefix=$PREFIX  --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++"
make -j4 install
cd ..
# A test will fail to build here but it should not matter.

apt-get source libyaml-cpp-dev
cd yaml-cpp-0.5.1/
cmake -DCMAKE_INSTALL_PREFIX:PATH=$PREFIX . && make all install
cd ..

apt-get source libsnappy-dev
cd snappy-1.1.0/
./configure --prefix=$PREFIX  --enable-static --without-gflags LDFLAGS="-L$PREFIX/lib -static -static-libstdc++"
make -j4 install
cd ..

cd aff4
./configure --prefix=$PREFIX  --enable-static LDFLAGS="-L$PREFIX/lib -L$PREFIX/libexec" --enable-static-binaries
make -j4 install
cd ..
```


## Building for windows.

The following instructions are for building windows binaries, on a Linux
system. On Ubuntu systems one must install the mingw platform first (using
`apt-get install g++-mingw-w64 mingw-w64`).



```
export PREFIX=/home/scudette/build/mingw/

apt-get source zlib1g
cd zlib-1.2.8.dfsg/
make -f win32/Makefile.gcc PREFIX=i686-w64-mingw32-
make -f win32/Makefile.gcc install INCLUDE_PATH=$PREFIX/include LIBRARY_PATH=$PREFIX/lib BINARY_PATH==$PREFIX/bin
cd ..

apt-get source libraptor2-dev
cd raptor2-2.0.13/
./configure --prefix=$PREFIX --enable-static --without-www LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 --enable-parsers="turtle ntriples" --enable-serializers="turtle ntriples"
make -j4 install
cd ..

apt-get source pcre3
cd pcre3-8.31/
./autogen.sh
./configure --prefix=$PREFIX --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CFLAGS="-static -mnop-fun-dllimport" --disable-cpp
cp /usr/bin/libtool ./libtool
make -j4 install
cd ..

apt-get source libpcre++-dev
cd libpcre++-0.9.5/
./autogen.sh
cp /usr/bin/libtool ./libtool
./configure --prefix=$PREFIX --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CXXFLAGS="-static -mnop-fun-dllimport -DPCRE_STATIC" --with-pcre-include="$PREFIX/include/"
make -j4 install
cd ..

apt-get source gflags
cd gflags-2.0/
touch src/port.h
./configure --prefix=$PREFIX --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CFLAGS="-static -mnop-fun-dllimport"
make -j4 install
cd ..


apt-get source  libgoogle-glog-dev
cd google-glog-0.3.3/
./configure --prefix=$PREFIX --enable-static --disable-shared LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CFLAGS="-static -mnop-fun-dllimport" CXXFLAGS="-I$PREFIX/include/ -Isrc/ -Isrc/windows/ -mnop-fun-dllimport -static -DGOOGLE_GLOG_DLL_DECL=''" --disable-frame-pointers

# In src/config.h at end

#undef HAVE_UNISTD_H
#undef TEST_SRC_DIR

make -j4 install
cd ..

apt-get source libyaml-cpp-dev
cd yaml-cpp-0.5.1/
cmake -DCMAKE_INSTALL_PREFIX:PATH=$PREFIX . && make all install
cd ..

apt-get source libsnappy-dev
cd snappy-1.1.0/
./configure --prefix=$PREFIX --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CXXFLAGS="-static -mnop-fun-dllimport -I$PREFIX/include/"
make -j4 install
cd ..

apt-get source liburiparser-dev
cd uriparser-0.7.5/
./configure --prefix=$PREFIX --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CXXFLAGS="-static -mnop-fun-dllimport -I$PREFIX/include/" --disable-test --disable-doc
make -j4 install
cd ..

apt-get source libyaml-cpp-dev
cd yaml-cpp-0.5.1/
echo "
# the name of the target operating system
SET(CMAKE_SYSTEM_NAME Windows)

# which compilers to use for C and C++
SET(CMAKE_C_COMPILER i686-w64-mingw32-gcc)
SET(CMAKE_CXX_COMPILER i686-w64-mingw32-g++)
SET(CMAKE_RC_COMPILER i686-w64-mingw32-windres)

# here is the target environment located
SET(CMAKE_FIND_ROOT_PATH  /usr/i686-w64-mingw32 $PREFIX )

# adjust the default behaviour of the FIND_XXX() commands:
# search headers and libraries in the target environment, search
# programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
" > Toolchain-mingw32.cmake

cmake -DCMAKE_TOOLCHAIN_FILE=Toolchain-mingw32.cmake -DCMAKE_INSTALL_PREFIX:PATH=$PREFIX .
cd ..

cd aff4
./configure --prefix=$PREFIX --enable-static LDFLAGS="-L$PREFIX/lib -static -static-libstdc++" --host=i686-w64-mingw32 CXXFLAGS="-static -mnop-fun-dllimport -I$PREFIX/include/" --enable-static-binaries
make -j4 install
cd ..
```

On very old versions of windows the binary might fail with a message "unable to
find strerror_s in msvcrt.dll". This is because such old versions do not contain
the function strerror_s, but they do contain strerror. The easiest way to fix
this is to hexedit the import table of the binary - simply find the string
strerror_s and replace the _s with null bytes. This makes the same binary work
fine on older windows versions.
