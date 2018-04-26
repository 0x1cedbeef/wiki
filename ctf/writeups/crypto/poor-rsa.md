<!-- TITLE: Poor Rsa -->
<!-- SUBTITLE: AlexCTF: CR4: Poor RSA -->

# 問題
[ctftime](https://ctftime.org/task/3350)

# 解法

```python
>>> from Crypto.PublicKey import RSA
>>> from base64 import b64decode 
>>> pubfile = open('key.pub', 'r').read()
>>> ciphertext = open('flag.b64', 'r').read()
```

公開鍵を読み込む

```python 
>>> pubkey = RSA.importKey(keyfile)
>>> pubkey.n
833810193564967701912362955539789451139872863794534923259743419423089229206473091408403560311191545764221310666338878019
>>> pubkey.e
65537 
```

## 小さいn

$n$ が小さいことに気づく

```python 
>>> len(bin(n)[2:])
399
```

一般的に、**768bit** 以下の *modulus* は素因数分解が現実的な時間内で可能である
参考: [Cryptology ePrint Archive: Report 2010/006](https://eprint.iacr.org/2010/006)

これ以下の桁数であれば、誰かがすでに素因数分解済みである可能性がある  
よって、 factordb.com から探してみる

今回は、[factordb.py](https://github.com/ihebski/factordb)を用いた

```sh
% factordb 833810193564967701912362955539789451139872863794534923259743419423089229206473091408403560311191545764221310666338878019
___________              __              ________ ___.    
\_   _____/____    _____/  |_  __________\______ \_ |__  
 |    __) \__  \ _/ ___\   __\/  _ \_  __ \    |  \| __ \ 
 |     \   / __ \  \___|  | (  <_> )  | \/    `   \ \_\ '
 \___  /  (____  /\___  >__|  \____/|__| /_______  /___  /
     \/        \/     \/                         \/    \/ 
================> https://github.com/ihebski/factordb
===========================================> by S0ld1er | Bugs_Bunny 

Status :FF => Composite, fully factored

 n: 833810193564967701912362955539789451139872863794534923259743419423089229206473091408403560311191545764221310666338878019 
 length : 120 

 p: 863653476616376575308866344984576466644942572246900013156919 
 length : 60 

 q: 965445304326998194798282228842484732438457170595999523426901 
 length : 60 
```

というわけで、2つの素数 $p, q$ が判明した

```python 
>>> n = pubkey.n
>>> e = pubkey.e
>>> p = 863653476616376575308866344984576466644942572246900013156919
>>> q = 96544530432699819479828222884248473243845717059599952342690
```

ところで、RSAにおける *private exponent* $d$ は

$$ed \equiv 1 \mod \phi(n)$$

から以下のように導ける

```python
>>> import gmpy
>>> totient_n = (p - 1)*(q - 1)
>>> d = gmpy.invert(e, totient_n)
```

$$m = c^d \mod n$$
であるから、

```python
>>> c = int.from_bytes(b64decode(ciphertext), 'big')
>>> print(c)
546514681109897642377058885254740938209760605167931631034138331308564188002339494648530153228068817245276146038543125484
>>> m = pow(c, d, n)
>>> print(m)
103145807013978985540355910125640487400385927671574445477620708860454905704702407706197052505510447088948250826538250
>>> len(str(hex(int(m)))[2:])
97
>>> b')\xe2m\xe6J\tb4\x88\xa7Sd\xcd\xba\xe8H\x93\xa0\x04\x14\xc4U\x845Dg\xb54\xd4\x14\xc4\xc5\xf5\x05$\x94\xd4U5\xf4\x15$U\xf4$\x14G\xd0'
```

なぜかうまくできない

## 別のアプローチ: 秘密鍵生成

うまくいかなかったので別のアプローチを取ってみる


$n, e, d, p, q$ がそろったので、ここからRSA private keyが生成できる  
ただし、`RSA.construct`に対して、**5-タプル**として引数を与えることと、 $d$ を`int`型にキャストすることに注意

```python 
>>> privatekey = RSA.construct((n, e, int(d), p, q))
>>> privatekey.exportKey().decode()
'-----BEGIN RSA PRIVATE KEY-----\nMIH5AgEAAjJSqZ4knufPPAy/ljoAlmF3K8nN9uHj+/xuRKB6Xg+JRFep+Bw64TKs\nVoPTWyi6XDJCQwIDAQABAjIzrQnKBvUPnpCxrK5x85DWuS8dbTtmFP+HEYHE3wja\nTF9QEkV6ZDCUBers1jQeQwJ5MQIaAImWgwYMdrnA3lgaaeDqnZG+0Qcb6x2SSjcC\nGgCZzedK7e6Hrf/daEy8R451mHC08gaS9lJVAhlmZEB1y+i/LC1L27xXycIhqKPe\naoR6qVfZAhlbPhKLmhFavne/AqQbQhwaWT/rqHUL9EMtAhk5pem+TgbW3zCYF8v7\nj0mjJ31NC+0sLmx5\n-----END RSA PRIVATE KEY-----'
>>> privatekey.decrypt(b64decode(ciphertext))
b'\x02\x9e&\xded\xa0\x96#H\x8au6L\xdb\xae\x84\x89:\x00ALEXCTF{SMALL_PRIMES_ARE_BAD}\n'
```

最初の方にゴミがついていて解読を邪魔していた模様  
したがって答えは

**ALEXCTF{SMALL_PRIMES_ARE_BAD}**