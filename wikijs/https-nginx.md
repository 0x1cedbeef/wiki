<!-- TITLE: Wiki.jsをNginxをリバースプロキシにしてhttps化 -->
<!-- SUBTITLE: ワイルドカード証明書ってちょっとかゆいところに手が届かなくありません？-->

# NginxとLet's encryptでリバースプロキシを実現

[公式wikiのページ ](https://docs.requarks.io/wiki/admin-guide/setup-nginx)にもnginxをリバースプロキシとしてhttpsでウェブに公開する手順が載っているが、大分ハマってしまったので、ここに手順を残す

## SSL証明書の取得（Let's encrypt）

[Let's encrypt](https://letsencrypt.org/)は最近ワイルドカード証明書を発行してくれるようになった。[^100][^101]


以下、サーバのOSを*Ubuntu 16.04*を前提として書く

まず、証明書を取得するためのパッケージ（*certbot*）をインストールする

```sh
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```

参考: [Nginx on Ubuntu 16.04 (xenial)](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)







## 参考リンク
[^100]: [ACME v2 and Wildcard Certificate Support is Live](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579)
[^101]: [ワイルカードSSLサーバ証明書とは](https://www.websecurity.symantec.com/ja/jp/theme/ssl-wildcard)
[Let's Encryptのワイルドカード証明書を早速発行してもらう](https://narusejun.com/archives/23/)
