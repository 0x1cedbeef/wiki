<!-- TITLE: Wiki.jsをNginxをリバースプロキシにしてhttps化 -->
<!-- SUBTITLE: ワイルドカード証明書ってちょっとかゆいところに手が届かなくありません？-->

# NginxとLet's encryptでリバースプロキシを実現

[公式wikiのページ ](https://docs.requarks.io/wiki/admin-guide/setup-nginx)にもnginxをリバースプロキシとしてhttpsでウェブに公開する手順が載っているが、大分ハマってしまったので、ここに手順を残す

## SSL証明書の取得（Let's encrypt）

[Let's encrypt](https://letsencrypt.org/)は最近ワイルドカード証明書を発行してくれるようになった。
参考1: [ACME v2 and Wildcard Certificate Support is Live](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579)
参考2[ワイルカードSSLサーバ証明書とは](https://www.websecurity.symantec.com/ja/jp/theme/ssl-wildcard)

以下、サーバのOSを*Ubuntu 16.04*を前提として書く

まず、CLIから証明書を取得するためのパッケージをインストールする

```sh
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```





## 参考リンク

[Let's Encryptのワイルドカード証明書を早速発行してもらう](https://narusejun.com/archives/23/)
