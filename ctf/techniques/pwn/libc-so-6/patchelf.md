<!-- TITLE: patchelf -->
<!-- SUBTITLE: A quick summary of patchelf -->

# `patchelf`によるglibcの変更

[\[Linux\] Multiple glibc libraries on a single host gcc | CODE Q&A \[English\]](https://code.i-harness.com/en/q/ced4b)

*patchelf* [^10] はすでにビルドされたELF形式実行ファイルの**動的ライブラリのサーチパス (rpath) **と**動的リンカ**を、任意のパスのものに変更することができる
バイナリビルド後に、[このページ](/ctf/techniques/pwn/libc-so-6/compile-glibc)でコンパイルした動的ライブラリおよび動的リンカを使うように変更してみる

# インストール
Ubuntu 16.04の場合、aptパッケージからインストール可能 (DebianやUbuntu 18.04では未確認)

```console
$ sudo apt update && sudo apt install -y patchelf
$ patchelf --version
patchelf 0.9
```

それ以外の場合は手動でインストールする
公式ページ [^10] の**Download**のヘッダから最新版のソースを入手し、あとは適当なパスでビルドする [^20]


# x86_64の場合

## ビルド済みのバイナリの情報

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
*N.B. 前者のコマンドは、動的ライブラリの* ***サーチパス*** *を表示することに注意*

```console
$ patchelf --print-rpath ./hello64

$ patchelf --print-interpreter ./hello64
/lib64/ld-linux-x86-64.so.2
```

特に指定しなければ、rpathは存在しないので、前者のコマンドは空白が表示されている

## rpathと動的リンカの指定

`--set-rpath`で動的ライブラリのサーチパスを設定可能

```console
$ patchelf --set-rpath $HOME/libc64/2.27/lib ./hello64
$ patchelf --print-rpath ./hello64
/home/vagrant/libc64/2.27/lib
```

`--print-rpath`で新たに設定されたrpathを確認できる

同時に、`--set-interpreter`で動的リンカのパスを設定する

```console
$ patchelf --set-interpreter $HOME/libc64/2.27/lib/ld-2.27.so ./hello64
$ patchelf --print-interpreter ./hello64
/home/vagrant/libc64/2.27/lib/ld-2.27.so
```

`ldd`コマンドで共有オブジェクトの依存関係を調べると && ELFバイナリを実行すると、

```console 
$ ldd ./hello64
	linux-vdso.so.1 =>  (0x00007ffcc9ee5000)
	libc.so.6 => /home/vagrant/libc64/2.27/lib/libc.so.6 (0x00007fe4ad2c6000)
	/home/vagrant/libc64/2.27/lib/ld-2.27.so => /lib64/ld-linux-x86-64.so.2 (0x00007fe4ad67b000)
$ ./hello64
Hello, World!
```

動的ライブラリと動的リンカが任意のものに変更され、かつ期待通りの実行結果を返すことが確認できた


# x86の場合

x86_64とほぼ同じなので割愛する
異なる点は、32ビットELFバイナリ用にlibcをコンパイルしておくことと、それに伴って指定するrpathやinterpreterのパスが変わることである


[^10]: [NixOS/patchelf: A small utility to modify the dynamic linker and RPATH of ELF executables](https://github.com/NixOS/patchelf)

[^20]: [PatchELFのビルド](https://qiita.com/komeda-shinji/items/6a86a0b466a67ddf3f47)