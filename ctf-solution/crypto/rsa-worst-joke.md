<!-- TITLE: Rsa Worst Joke -->
<!-- SUBTITLE: A quick summary of Rsa Worst Joke -->

# 準備

ファイルを読み込む

```python
pem = open('public.pem', 'r')
ciphertext = open('flag.enc', 'r')
```

`pem`をRSA公開鍵として読み込む  
modulus **n** と、public exponent **e** も同時に代入する

```python
from Crypto.PublicKey import RSA
pubkey = RSA.importKey(pem)
n = pubkey.n
e = pubkey.e
```


φ(n)は本来2つの素数 **p**. **q** より `(p-1) * (q-1)`で計算されるが、今回は **n** を構成するのに、ただ1つの素数 **p** しか用いていない  
したがって、` n == p `より
```
φ(n) = φ(p)
     = p - 1
     = n - 1
```

Pythonでは

```python
totient_n = n - 1
```

# *ciphertext*の数値変換
次に *ciphertext* を数値変換する  
*ciphertext*はbase64エンコードされていることから、

```python
from base64 import b64decode
b64decodedCipher = b64decode(ciphertext)
```

この*b64decodedCipher*は*byte*型となる  
*byte*型を*int*型にするには、***int.from_bytes***メソッドを使う

```python
c = int.from_bytes(b64decodedCipher, 'big')
```

第2引数は本来エンディアンネスを表すものだが、今回の場合先頭のバイトから順に*int*変換していくので*big endian*として処理させる

# d (private exponent)を求める

RSAで*d*は以下のように与えられる


```
e*d ≡ 1 (mod φ(n))
```

先程すでに**e**と**n**を求めているので、**d**はモジュール*gmpy*を用いて以下のようにして求められる

```python 
import gmpy
d = gmpy.invert(e, totient_n)
```


*gmpy*を使う際にはあらかじめaptパッケージをインストールする必要がある

```shell
$ sudo apt update && sudo apt install python3-dev limgmp3-dev
$ sudo -H pip3 install gmpy
```

<a href=https://code.google.com/p/gmpy/source/browse/trunk/doc/gmpydoc.txt>gmpyのdocumentation</a>


# 復号、そして暗号化

RSAの暗号化はと復号は、

```python
c = m ^ e mod n # encryption
m = c ^ d mod n # decryption
```

Pythonの**pow**関数は第3引数を取ると、`{第1引数}^{第2引数} % {第3引数}`を返すようになっている

```python
m = pow(c, d, n)
```

これで復号された数値が取得できたので、これを文字列に変換する

```python
message = bytes.fromhex(str(hex(int(m)))[2:]).decode('utf8')
```

# スクリプト

```python
from Crypto.PublicKey import RSA
from base64 import b64decode
import gmpy

with open('public.pem', 'r') as f:
    pem = f.read()
with open('flag.enc', 'r') as f:
    ciphertext = f.read()

# import a public key
pubkey = RSA.importKey(pem)
# modulus 
n = pubkey.n
# φ(n) = φ(p) = p-1 = n-1
totient_n = n - 1
# public exponent, 65537
e = pubkey.n

# ciphertext in numerical values
c = int.from_bytes(b64decode(ciphertext), 'big')

# private exponent is easily obtained in this case
d = gmpy.invert(e, totient_n)

m = pow(c, d, n)
message = bytes.fromhex(str(hex(int(m)))[2:]).decode('utf8')

print(message)
```

```shell
$ python3 solve.py
The empire secret system has been exposed ! The top secret flag is : Flag{S1nGL3_PR1m3_M0duLUs_ATT4cK_TaK3d_D0wn_RSA_T0_A_Sym3tr1c_ALg0r1thm}
```