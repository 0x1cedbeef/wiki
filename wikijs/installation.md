<!-- TITLE: Wiki.js インストール -->
<!-- SUBTITLE: インストール時に詰まったときに見るページ -->

# はじめに

Wiki.jsについては公式ドキュメントを読んだほうが早い  
ここではインストール時に詰まったことを列挙する

## UIがおかしい

よくよく[Troubleshootingのドキュメント](https://docs.requarks.io/wiki/troubleshooting)を見ればわかることだが、`node wiki configure`時に、**Hostの項目にはポート番号も含めて入力する**ことを忘れずに行う

例えば *localhost* のポート *8080* なら、

```
Host: http://localhost
Port: 8080
```

ではなく

```
Host: http://localhost:8080
Port: 8080
```

のように記入する

