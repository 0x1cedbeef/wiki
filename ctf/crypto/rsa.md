<!-- TITLE: RSA -->
<!-- SUBTITLE: RSA暗号の問題を解く上でのまとめ -->

# 概要

RSA暗号とは, 公開鍵暗号の一種であり, おそらく最も有名な公開鍵暗号.
Rivest, Shamir, Adlemanの三人の発明者の頭文字を取って名付けられた.

# 暗号化手順

まず, 暗号化したい平文のメッセージを $m$ とおく.
一般に、この $m$ は数値で表される. もし, 他の種類のメッセージ（例えばASCII文字からなる英文）を暗号化したいならば、あらかじめ数値に変換する必要がある。

以下にPython3による英文の数値変換例を示す.

```python
>>> s = 'Hello, World!'
>>> s.encode()
b'Hello, World!'
>>> int.from_bytes(s.encode(), 'big')
5735816763073854918203775149089
```

文字列 $s$ に対し, まず[str.encode()](https://docs.python.org/3/library/stdtypes.html#str.encode)でbytes型に変換して、その結果を数値（int型）に[int.from_bytes()](https://docs.python.org/3/library/stdtypes.html#int.from_bytes)を用いて変換している.
*int.from_bytes*の第2引数の*'big'* は, *big endian*を意味していて, バイト列をそのまま順番に先頭から処理して先頭から格納することを示している. 
（あえて言うなら, *little endian*として処理しないということでもある. 補足として*big endian*で処理したときと, *little endian*で処理したときの違いを示す）
```python
>>> hex(int.from_bytes('ABCDEFGH'.encode(), 'big'))
'0x4142434445464748'
>>> hex(int.from_bytes('ABCDEFGH'.encode(), 'little'))
'0x4847464544434241'
```


