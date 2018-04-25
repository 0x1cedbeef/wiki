<!-- TITLE: Wiki.jsをNginxをリバースプロキシにしてhttps化 -->
<!-- SUBTITLE: ワイルドカード証明書ってちょっとかゆいところに手が届かなくありません？-->

# NginxとLet's encryptでリバースプロキシを実現

[公式wikiのページ ](https://docs.requarks.io/wiki/admin-guide/setup-nginx)にもnginxをリバースプロキシとしてhttpsでウェブに公開する手順が載っているが、大分ハマってしまったので、ここに手順を残す
以下、サーバのOSを*Ubuntu 16.04*を前提として書く

## ドメインの取得とDNSの設定

まず今までWiki.jsをlocalhostで使ってきたなら、今ここでドメインの取得が必要になる
お金をかけたくないなら、*.tk, .ga, .cf* などは[freenom](https://www.freenom.com/en/index.html)から無料で取得できる

同時に、

Freenomの場合ドメイン登録時には、画像のようにサーバのDNSなど、外部のDNSサーバを指定すること
例えばDigitaloceanならば、プライマリには `ns1.digitalocean.com`、セカンダリには `ns2.digitalocean.com`を設定する

![Freenom](/uploads/img/freenom.png "Freenom")

登録が終わったら、今度はサーバのコンパネからDNSの設定を行う（サーバ会社がAPIを用意してあって、それを使うならばこの限りではない）
Digitaloceanの場合は **Networking &rarr; Domains**から新しく設定をする

![Digitalocean 01](/uploads/img/Digitalocean01.png "Digitalocean 01")



## SSL証明書の取得（Let's encrypt）

[Let's encrypt](https://letsencrypt.org/)は最近ワイルドカード証明書を発行してくれるようになった。[^100] [^101]




まず、証明書を取得するためのパッケージ（*certbot*）をインストールする

```sh
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```

参考: [Nginx on Ubuntu 16.04 (xenial)](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)


[^200] を参考にして、ワイルドカード証明書の発行をしてもらう
なおこのコマンドはroot権限が必要

```sh 
$ sudo certbot certonly \
--manual \
--preferred-challenges dns-01 \
--server https://acme-v02.api.letsencrypt.org/directory \
--domain *.yourdomain.com
```

メールアドレスを入力して、いま繋がっているIPアドレスを記録されることに同意すると、DNSのTXTレコードにこれを記録しろとの記述が出る

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.wiki.0x1cedbeef.cf with the following value:

eA…（中略）…mc

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
```


## 参考リンク
[^100]: [ACME v2 and Wildcard Certificate Support is Live](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579)
[^101]: [ワイルカードSSLサーバ証明書とは](https://www.websecurity.symantec.com/ja/jp/theme/ssl-wildcard)
[^200]: [Let's Encryptのワイルドカード証明書を早速発行してもらう](https://narusejun.com/archives/23/)
