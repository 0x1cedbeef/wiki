<!-- TITLE: Missing -->
<!-- SUBTITLE: A quick summary of Missing -->

# `gef missing`のアレ

```console
gef➤  gef missing
[*] Command `assemble` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
[*] Command `ropper` is missing, reason  →  Missing `ropper` package for Python3, install with: `pip3 install ropper`.
[*] Command `unicorn-emulate` is missing, reason  →  Missing `unicorn` package for Python3. Install with `pip3 install unicorn`.
[*] Command `set-permission` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
[*] Command `capstone-disassemble` is missing, reason  →  Missing `capstone` package for Python3. Install with `pip3 install capstone`.
```

そのまま`pip3`でインストールすると **module 'keystone' has no attribute 'KS_ARCH_X86'** というエラーでうまくいかなくなるので、一部手動でインストール

```console
$ sudo apt update && sudo apt install -y build-essential python3 python3-dev python3-pip gdb libcapstone3 libcapstone-dev cmake
$ sudo -H pip3 install unicorn capstone filebytes
$ wget https://github.com/keystone-engine/keystone/archive/0.9.1.tar.gz
$ tar xzvf 0.9.1.tar.gz 
$ cd keystone-0.9.1/
$ mkdir build
$ cd build
$ ../make-share.sh
$ sudo make install
$ sudo ldconfig
$ kstool 
$ cd ../bindings/python/
$ sudo make install3
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

# 参考

[Download – Keystone – The Ultimate Assembler](http://www.keystone-engine.org/download/)

[sashs/Ropper: Display information about files in different file formats and find gadgets to build rop chains for different architectures (x86/x86_64, ARM/ARM64, MIPS, PowerPC). For disassembly ropper uses the awesome Capstone Framework.](https://github.com/sashs/Ropper)

(https://www.capstone-engine.org/documentation.html)