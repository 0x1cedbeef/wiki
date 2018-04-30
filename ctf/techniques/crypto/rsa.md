<!-- TITLE: RSA -->
<!-- SUBTITLE: RSA暗号の問題を解く上でのまとめ -->

# 概要

RSA暗号とは, 公開鍵暗号の一種であり, おそらく最も有名な公開鍵暗号.
Rivest, Shamir, Adlemanの三人の発明者の頭文字を取って名付けられた.

# 暗号化手順

## メッセージ *m*

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

または、*binascii*を用いて変換することもできる

```python
>>> import binascii
>>> s = 'Hello, World!'
>>> s.encode()
b'Hello, World!'
>>> binascii.hexlify(s.encode())
b'48656c6c6f2c20576f726c6421'
>>> int('0x' + binascii.hexlify('Hello, World!'.encode()).decode('utf8'), 16)
5735816763073854918203775149089
```

文字列 $s$ に対し, まず[str.encode()](https://docs.python.org/3/library/stdtypes.html#str.encode)でbytes型に変換して、その結果を数値（int型）に[int.from_bytes()](https://docs.python.org/3/library/stdtypes.html#int.from_bytes)を用いて変換している.
*int.from_bytes*の第2引数の *'big'* は, *big endian*を意味していて, バイト列をそのまま順番に先頭から処理して先頭から格納することを示している. 
（あえて言うなら, *little endian*として処理しないということでもある. 補足として*big endian*で処理したときと, *little endian*で処理したときの違いを示す）
```python
>>> hex(int.from_bytes('ABCDEFGH'.encode(), 'big'))
'0x4142434445464748'
>>> hex(int.from_bytes('ABCDEFGH'.encode(), 'little'))
'0x4847464544434241'
```

（未完）

## モジュラス *n* と二つの素数

まず二つの素数 $p, q$ を2進数で表したときの桁数（すなわちビット数）を定める。  
これら二つの素数をかけ合わせたものが $n$ となる
したがって $$n = pq$$
UN\*Xコマンドの`ssh-keygen`でRSA暗号方式の鍵を作った場合、桁数を指定しなければ2048bitの $n$ を生成するようになっている
$p, q$ のビット数は同じだから、この場合二つの素数はともに1024bitである
**N.B.** 1024bitの数には、*0* から始まるものも含まれることに注意（例えば *00....0010* （10進数で *2*）も1024bitの素数として扱われる）
なお、 *-b* フラグでビット数を指定した場合はこの限りではなく、ビット数を変えられる（このコマンドの場合は1024bit以上）
ここではこのコマンドを用いずに、他の方法で素数を生成する
TODO

## public exponent *e*


## private exponent *d*


## RSA暗号鍵のペア作成

ここでは、あえて`ssh-keygen`を用いずに、python3のモジュール **[PyCrypto](https://pypi.org/project/pycrypto/)** でRSA暗号鍵のペアを作成する

```python
>>> p = {素数p}
>>> q = {素数q}
>>> n = p*q
>>> totient_n = (p - 1)*(q - 1)
>>> message = 'Call me Ishmael.'
>>> m = int.from_bytes(s.encode(), 'big')
>>> 