<!-- TITLE: gef missingの解決 -->
<!-- SUBTITLE: A quick summary of Missing -->

# `gef missing`のアレ

```console
gef>  gef missing
[*] Command `assemble` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
[*] Command `ropper` is missing, reason  →  Missing `ropper` package for Python3, install with: `pip3 install ropper`.
[*] Command `unicorn-emulate` is missing, reason  →  Missing `unicorn` package for Python3. Install with `pip3 install unicorn`.
[*] Command `set-permission` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
[*] Command `capstone-disassemble` is missing, reason  →  Missing `capstone` package for Python3. Install with `pip3 install capstone`.
```

そのまま`pip3`でインストールすると **module 'keystone' has no attribute 'KS_ARCH_X86'** というエラーでうまくいかなくなるので、一部手動でインストールする
Python2でも適宜読み替えて解決できる

## unicorn, capstone (依存パッケージ)

まずは依存パッケージ [^10]

```console
$ sudo apt update && sudo apt install -y build-essential python3 python3-dev python3-pip gdb libcapstone3 libcapstone-dev cmake
$ sudo -H pip3 install unicorn capstone filebytes
```

## keystone

*keystone*をソースからビルド

```console
$ wget https://github.com/keystone-engine/keystone/archive/0.9.1.tar.gz
$ tar xzvf 0.9.1.tar.gz 
$ cd keystone-0.9.1/
$ mkdir build
$ cd build
$ ../make-share.sh
$ sudo make install
```

`sudo ldconfig`で共有ライブラリの依存情報を更新して、`kstool`コマンドが使えることを確認する

```console
$ sudo ldconfig
$ kstool 
```

pythonバインディングをインストールする
*Python2*用にインストールするならば、代わりに`sudo make install`を実行する

```console
$ cd ../bindings/python/
$ sudo make install3
```

## ropper

最後に*ropper*をインストールする

```console
$ sudo -H pip3 install ropper
```

# 確認

```console
$ gdb -q
GEF for linux ready, type `gef' to start, `gef config' to configure
67 commands loaded for GDB 7.11.1 using Python engine 3.5
gef>  gef missing
[+] No missing command
```

# 参考リンク

[Download – Keystone – The Ultimate Assembler](http://www.keystone-engine.org/download/)

[keystone/COMPILE-NIX.md at master · keystone-engine/keystone](https://github.com/keystone-engine/keystone/blob/master/docs/COMPILE-NIX.md)

[Ropper/README.md at master · sashs/Ropper](https://github.com/sashs/Ropper/blob/master/README.md)

[Documentation – Capstone – The Ultimate Disassembler](https://www.capstone-engine.org/documentation.html)

[^10]: Ubuntu 16.04のcmakeはバージョンが[最新バージョン](https://cmake.org/download/)より遅れているので、他に使うことも考えてソースからビルドしたほうがいいかもしれないですね
