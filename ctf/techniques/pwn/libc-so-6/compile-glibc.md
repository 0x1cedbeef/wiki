<!-- TITLE: Compile Glibc -->
<!-- SUBTITLE: glibcの手動コンパイル方法 -->

# ソース取得
- [ここ](https://ftp.gnu.org/gnu/libc/)からコンパイルするバージョンのtar.gzファイルを取得する
- 今回は執筆時点での最新版、2.27を用いる
- glibcのコンパイルに使うディレクトリは、解凍されたソースディレクトリとは異なるものを用いる必要があるので、別に作ってやる

# コンパイル

## x86_64 

まずx86_64向け


```console 
$ wget --quiet 'https://ftp.gnu.org/gnu/libc/glibc-2.27.tar.gz'
$ tar xzvf glibc-2.27.tar.gz
$ mkdir glibc-2.27-build-64 && cd glibc-2.27-build-64
$ mkdir -p $HOME/libc64/2.27
$ ../glibc-2.27/configure --prefix=$HOME/libc64/2.27 
$ make -j8 && make install 
```

自分の環境ではmakeするのに5分ほどかかった
コンパイルされたlibcを用いてリンクするには、例えば以下のようにする [^10]


```console 
$ cat hello.c
#include <stdio.h>

int main(void) {
  printf("Hello, World!\n");
  return 0;
}
$ gcc hello.c -o hello \
  -Wl,--rpath=$HOME/libc64/2.27/lib/libc.so.6 \
  -Wl,--dynamic-linker=$HOME/libc64/2.27/lib/ld-2.27.so
$ ldd ./hello
	linux-vdso.so.1 =>  (0x00007ffef2912000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4fafcde000)
	/home/vagrant/libc64/2.27/lib/ld-2.27.so => /lib64/ld-linux-x86-64.so.2 (0x00007f4fb00a8000)
$ ./hello
Hello, World!
```

## x86 (i686)

ここ [^20] を参考にコンパイルする
なお、*i686*部分を*i386*に変えて`configure`しても、**deprecated**だと言われてしまうので、このままコンパイルする

```console 
$ mkdir -p $HOME/libc32/2.27
$ ../glibc-2.27/configure --prefix=$HOME/libc32/2.27 \
     --host=i686-pc-linux-gnu \
     --build=i686-pc-linux-gnu \
     CC="gcc -m32" CXX="g++ -m32" \
     CFLAGS="-O2 -march=i686" \
     CXXFLAGS="-O2 -march=i686"
$ make -j8 && make install 
```

# 参考リンクなど
[Using the GNU Compiler Collection (GCC): x86 Options](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)


<!-- annotations -->

[^10]: [linux - Multiple glibc libraries on a single host - Stack Overflow](https://stackoverflow.com/a/851229/8501077)

[^20]: [linux - How to compile glibc 32bit on an x86_64 machine - Stack Overflow](https://stackoverflow.com/a/8074427/8501077)




