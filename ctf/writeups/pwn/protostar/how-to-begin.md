<!-- TITLE: How To Begin -->
<!-- SUBTITLE: A quick summary of How To Begin -->

# はじめ方

Exploit challengeの2つ目のVM
中身は*Debian Squeeze*であり、VirtualBoxで起動の際は **New &rarr; Linux/Ubuntu (32-bit) または Linux/Debian (32-bit)** を選ぶ

GUI(X window)はないので、メモリはそこまで充てなくとも良い

起動するとログインを求められる
ID: *user*, PW: *user* でログイン

```sh
protostar login: user
Password: user
```

## ssh設定

このままVM上で進めても良いが、sshでログインすると楽なのでぜひやろう

[^10] より、ローカルで以下のコマンド
(ポート番号3022は使えるなら他の番号でも構わない)

```sh
(local) $ VBoxManage modifyvm {Protostar_vm_name} --natpf1 "ssh,tcp,,3022,,22"
(local) $ VBoxManage showvminfo {Protostar_vm_name} | grep 'Rule' 
NIC 1 Rule(0):   name = ssh, protocol = tcp, host ip = , host port = 3022, guest ip = , guest port = 22
(local) $ ssh user@127.0.0.1 -p 3022
```

ついでに`~/.ssh/config`に追記

```
Host Protostar
HostName 127.0.0.1 
User user
Port 3022
UserKnownHostsFile /dev/null
```

sshでログイン

```sh
(local) $ ssh Protostar 
The authenticity of host '[127.0.0.1]:3022 ([127.0.0.1]:3022)' can't be established.
RSA key fingerprint is SHA256:*******************************************.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:3022' (RSA) to the list of known hosts.


    PPPP  RRRR   OOO  TTTTT  OOO   SSSS TTTTT   A   RRRR  
    P   P R   R O   O   T   O   O S       T    A A  R   R 
    PPPP  RRRR  O   O   T   O   O  SSS    T   AAAAA RRRR  
    P     R  R  O   O   T   O   O     S   T   A   A R  R  
    P     R   R  OOO    T    OOO  SSSS    T   A   A R   R 

          http://exploit-exercises.com/protostar                                                 

Welcome to Protostar. To log in, you may use the user / user account.
When you need to use the root account, you can login as root / godmode.

For level descriptions / further help, please see the above url.

user@127.0.0.1's password: user
Linux (none) 2.6.32-5-686 #1 SMP Mon Oct 3 04:15:24 UTC 2011 i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed May  9 14:17:34 2018
$ 
```

`/bin/sh (/bin/dash)`だと使いづらいので、`/bin/bash`に`chsh`する

```sh
$ chsh -s /bin/bash user
Password: user
$ logout
```

再度ログイン

```bash
(local) $ ssh Protostar
⋮
(先ほどと同じなので省略)
⋮
Last login: Wed May  9 14:27:33 2018 from 10.0.2.2
user@protostar:~$ 
```

# 重要な情報

問題を解くにあたって、前もって設定することがいくつかある

## 問題の場所

`/opt/protostar/bin`

## rootになる方法

```bash
user@protostar:~$ su
Password: godmode
root@protostar:/home/user# whoami
root
```

なお`apt-get update`などは *Debian Squeeze* のサポート期限が切れているためデフォルトの設定では作動しない
`/etc/apt/sources.list`を編集すれば可能だが、問題を解く上では必要ない

## rootになってやるべきこと

デフォルトではcoredumpが生成されないようになっており、問題を解くうえで大きな障害になるので設定する

```sh
root@protostar:/home/user# sysctl -a | grep suid_dumpable
fs.suid_dumpable = 0
root@protostar:/home/user# sysctl -w fs.suid_dumpable=2
fs.suid_dumpable = 2
root@protostar:/home/user# exit
exit
user@protostar:~$ ulimit -a | grep core
core file size          (blocks, -c) 0
user@protostar:~$ ulimit -c unlimited
user@protostar:~$ ulimit -a | grep core
core file size          (blocks, -c) unlimited
```

これでcoredumpが生成される
ただし、生成されるパスが

```bash
user@protostar:~$ cat /proc/sys/kernel/core_pattern
/tmp/core.%s.%e.%p
```

のように、`/tmp`以下なので注意すること