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
なおこのコマンドはroot権限が必要で、かつ証明書のパスが出力されるので、念のために`script`コマンドでログを取る
また`{youdomain.com}`部分を自分のドメインに書き換えること
例えば`wiki.example.com`なら、`--domain *.example.com`とする

```sh 
$ script
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

*TXTレコード*をDNSの設定に追加したら、ターミナルに戻って **Enter**キーを押す
すると、以下のような証明書の作成に成功したとの表示とが出る

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/{yourdomain.com}/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/{yourdomain.com}/privkey.pem
   Your cert will expire on 20xx-xx-xx. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## Nginxのリダイレクトの設定

まずNginxを一度止める

```sh
$ sudo systemctl stop nginx
```

ディレクトリ`/etc/nginx/sites-available`に、`{yourdomain.com}.conf`という名前で設定ファイルを作成して編集する

```sh
$ sudo vim /etc/nginx/sites-available/{yourdomain.com}.conf
```

内容は、[^300] を参考にし以下のようにする
なお、`localhost:8080`で**Wiki.js**が動いているものとする

```
server {
  listen      [::]:80 ipv6only=off;
  server_name {wiki.yourdomain.com};
  return      301 https://$server_name$request_uri;
}
server {
  listen      443 ssl http2;
  listen      [::]:443 ssl http2;
  server_name {wiki.yourdomain.com};

  ssl_session_timeout 1d;
  ssl_session_cache   shared:SSL:50m;
  ssl_session_tickets off;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 $
  ssl_prefer_server_ciphers on;

  ssl_certificate     /etc/letsencrypt/live/{yourdomain.com}/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/{yourdomain.com}/privkey.pem;

  location / {
    proxy_set_header    Host $http_host;
    proxy_set_header    X-Real-IP $remote_addr;
    proxy_pass          http://127.0.0.1:8080;
    proxy_http_version  1.1;
    proxy_set_header    Upgrade $http_upgrade;
    proxy_set_header    Connection "upgrade";
    proxy_next_upstream error timeout http_502 http_503 http_504;
  }
}
```

書き終わったら、`/etc/nginx/sites-enabled`にsymlinkを貼って、デフォルトで置いてあるsymlinkを削除する

```sh
$ cd /etc/nginx/sites-enabled
$ sudo ln -s /etc/nginx/sites-available/{yourdomain.com}.conf .
$ unlink ./default
$ ls -l
total 0
lrwxrwxrwx 1 root root 45 Xxx xx xx:xx {yourdomain}.conf -> /etc/nginx/sites-available/{yourdomain.com}.conf
```


## Wiki.jsの再設定

ここまで来れば、あとは**Wiki.js**を再設定してNginxを再起動するだけ

```sh
$ cd /path/to/wiki
$ node wiki stop && node wiki configure 
```

ここのアドレスとポートの設定で四苦八苦したが、結局のところこうすればよい

![Port And Host](/uploads/img/port_and_host.png "Port And Host")

Hostに`{wiki.yourdomain.com}:443`
Portに`8080`を設定するとうまくいく

あとは前と同じように設定をして、**Wiki.js**をスタートさせる
ターミナルにスタートしたとの表示が出たら、`http://{wiki.yourdomain.com}`にアクセスして、`https://{wiki.yourdomain.com}`にリダイレクトされることを確認する

## さいごに

お疲れ様でした
Let's encryptの証明書は90日で無効になってしまうので、自動化しておくといいと思います

## 参考リンク
[^100]: [ACME v2 and Wildcard Certificate Support is Live](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579)
[^101]: [ワイルカードSSLサーバ証明書とは](https://www.websecurity.symantec.com/ja/jp/theme/ssl-wildcard)
[^150]: [Nginx on Ubuntu 16.04 (xenial)](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)
[^200]: [Let's Encryptのワイルドカード証明書を早速発行してもらう](https://narusejun.com/archives/23/)
[^300]: [Installing Wiki.js on Ubuntu 16.04](https://www.theo-andreou.org/?p=1744)