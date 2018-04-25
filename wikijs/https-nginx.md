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

ここから *Aレコード*を追加する
ここではのちにワイルドカード証明書を使うことを考慮して、`example.com` のようなルートドメインではなく、`wiki.example.com`のようにサブドメインを指定する
また、*TTL (Time To Live)*はデフォルトのままで構わない

*Aレコード*の追加が終わったら、ドメインから正しくサーバに飛べるか確認をする

```sh
$ sudo systemctl start nginx 
```

初期状態ならnginxは、ディレクトリ`/etc/nginx/sites-enabled`のなかにsymlink`default`が入ったままなので、この状態で設定したドメインにアクセスすると、テストページが表示されるはずである
なお、ここからのページ表示の確認作業は、キャッシュされたページの表示を防ぐためにも**ブラウザのシークレットモード**でアクセスすることをおすすめする

## SSL証明書の取得（Let's encrypt）

[Let's encrypt](https://letsencrypt.org/)は最近ワイルドカード証明書を発行してくれるようになった。[^100] [^101]
せっかくなので（？）*Let's encrypt*の証明書を使って設定してみる

まず、証明書を取得するためのパッケージ（*certbot*）をインストールする [^150]

```sh
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```


この記事[^200] を参考にして、ワイルドカード証明書の発行をしてもらう
なおこのコマンドはroot権限が必要
`{youdomain.com}`部分を自分のドメインに書き換えること
例えば`wiki.example.com`なら、`--domain *.example.com`とする

```sh 
$ sudo certbot certonly \
--manual \
--preferred-challenges dns-01 \
--server https://acme-v02.api.letsencrypt.org/directory \
--domain *.{yourdomain.com}
```

メールアドレスを入力して、いま繋がっているIPアドレスを記録されることに同意すると、DNSのTXTレコードにこれを記録しろとの記述が出る

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.{yourdomain.com} with the following value:

eA…（中略）…mc

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
```

この`eA…（中略）…mc`を、先程Aレコードを設定したサーバのコンパネまたはAPIから*TXTレコード*に登録する


![Acme Challenge](/uploads/img/acme_challenge.png "Acme Challenge")

## 参考リンク
[^100]: [ACME v2 and Wildcard Certificate Support is Live](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579)
[^101]: [ワイルカードSSLサーバ証明書とは](https://www.websecurity.symantec.com/ja/jp/theme/ssl-wildcard)
[^150]: [Nginx on Ubuntu 16.04 (xenial)](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)
[^200]: [Let's Encryptのワイルドカード証明書を早速発行してもらう](https://narusejun.com/archives/23/)
