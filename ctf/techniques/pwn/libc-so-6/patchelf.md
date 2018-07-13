<!-- TITLE: libc.so.6 -->
<!-- SUBTITLE: A quick summary of Libc So 6 -->

# `patchelf`によるglibcの変更

[\[Linux\] Multiple glibc libraries on a single host gcc | CODE Q&A \[English\]](https://code.i-harness.com/en/q/ced4b)

*patchelf* [^10] はすでにビルドされたELF形式実行ファイルの**動的ライブラリのサーチパス (rpath) **と**動的リンカ**を、任意のパスのものに変更することができる
バイナリビルド後に、[このページ](/ctf/techniques/pwn/libc-so-6/compile-glibc)でコンパイルした動的ライブラリおよび動的リンカを使うように変更してみる


# x86_64の場合

```console
$ cat hello.c 
#include <stdio.h>

int main(void) {
  printf("Hello, World!\n");
  return 0;
}
$ gcc hello.c -o hello64
$ ./hello64 
Hello, World!
$ ldd ./hello64
	linux-vdso.so.1 =>  (0x00007ffef49dc000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fce9b2d1000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fce9b69b000)
```

このように、システムのデフォルトの動的ライブラリと動的リンカが使われていることがわかる
これは`patchelf --print-rpath <filename>`と`patchelf --print-interpreter <filename>`で知ることもできる

```console
$ patchelf --print-rpath ./hello64

$ patchelf --print-interpreter ./hello64
/lib64/ld-linux-x86-64.so.2
```





```console
$ patchelf --set-rpath $HOME/libc64/2.27/lib ./hello64
$ patchelf --set-interpreter $HOME/libc64/2.27/lib/ld-2.27.so ./hello64
$ patchelf --print-rpath ./hello64
/home/vagrant/libc64/2.27/lib
$ patchelf --print-interpreter ./hello64
/home/vagrant/libc64/2.27/lib/ld-2.27.so
$ ./hello64
Hello, World!
$ ldd ./hello64
	linux-vdso.so.1 =>  (0x00007ffcc9ee5000)
	libc.so.6 => /home/vagrant/libc64/2.27/lib/libc.so.6 (0x00007fe4ad2c6000)
	/home/vagrant/libc64/2.27/lib/ld-2.27.so => /lib64/ld-linux-x86-64.so.2 (0x00007fe4ad67b000)

```


[^10]: [NixOS/patchelf: A small utility to modify the dynamic linker and RPATH of ELF executables](https://github.com/NixOS/patchelf)

