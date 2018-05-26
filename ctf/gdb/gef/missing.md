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

そのまま`pip3`でインストールすると**module 'keystone' has no attribute 'KS_ARCH_X86'**というエラーでうまくいかなくなるので、一部手動でインストール

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

