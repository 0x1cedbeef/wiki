<!-- TITLE: The Vault -->
<!-- SUBTITLE: A quick summary of The Vault -->

# The Vault - writeup

## 問題

[ctftime](https://ctftime.org/task/5676)

## 解法

```
（長々とした前置き）
But the door. It won't budge. It says it will answer only to the DUNGEON_MASTER.
Have you not shown your worth?
But more than that,
It demands to know your secrets.
（長々とした後置き）
http://chal1.swampctf.com:2694
```

つまりは`ID: DUNGEON_MASTER`として、ログインをしてくれという問題  
とりあえず脳死SQLインジェクションしてみるが意味なし

ソースの`<script>`タグ内に以下のような記述を発見

```javascript
<script>
function login(){
	var xhttp = new XMLHttpRequest();
	xhttp.onreadystatechange = function() {
		if (this.readyState == 4) {
			if(this.status == 200){
				alert(xhttp.responseText);
				window.location.href = 'https://youtu.be/rWVeZx2IP30?t=3';
			} else {
				alert('Invalid Credentials')
			}
		}
	};
	xhttp.open("POST", "/login/"+document.getElementById('inputName').value+"."+document.getElementById('inputPassword').value, true);
	xhttp.send();
	return false;
}
```

URL *`http://chal1.swampctf.com:2694/login/{id}.{pass}`* に正しくPOSTすれば勝利っぽいことがわかる  
idは **DUNGEON_MASTER** だとわかっているから、あとはpasswordだけ

とりあえず適当にPOSTしてみる

```python
>>> import requests
>>> s = requests.Session()
>>> url_base = 'http://chal1.swampctf.com:2694/'
>>> s.post(url_base + 'login/DUNGEON_MASTER.passwd')
<Response [500]>
>>> r = s.post(url_base + 'login/DUNGEON_MASTER.passwd')
>>> r.text
'<!DOCTYPE html>\n<html lang="en">\n<head>\n<meta charset="utf-8">\n<title>Error</title>\n</head>\n<body>\n<pre>test_hash [0d6be69b264717f2dd33652e212b173104b4a647b7c11ae72e9885f11cd312fb] does not match real_hash[40f5d109272941b79fdf078a0e41477227a9b4047ca068fff6566104302169ce]</pre>\n</body>\n</html>\n'
```

64文字 &rarr; 32byte &rarr; 256bitのhash値なのでおそらくSHA256と予想
hash値が違うよと言われているが、ご丁寧に正しいhash値 **40f5d109272941b79fdf078a0e41477227a9b4047ca068fff6566104302169ce** を教えてくれている

もうすでに前半部分はわかっているので、あとはデコーダーに入れて解く

[Hash Decoder](https://www.dcode.fr/hash-function)

```
HASH / FINGERPRINT
40f5d109272941b79fdf078a0e41477227a9b4047ca068fff6566104302169ce

ALGORITHM 
SHA256 (64 hex characters)

SALT PREFIXED HASH(SALT+WORD) 
DUNGEON_MASTER.
```

&uarr; **.** （ドット）を忘れずに

結果

```
Results

SHA256
smaug123
```

したがってクレデンシャルは
```
ID: DUNGEON_MASTER
PW: smaug123
```

これを入力してログインすると、ブラウザのダイアログに以下の文章が表示される

```
From chal1.swampctf.com:2694
flag{somewhere_over_the_rainbow_tables}
```

どうやら正規の解法ではRTを使ってほしかった模様