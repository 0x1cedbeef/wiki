<!-- TITLE: Https Nginx -->
<!-- SUBTITLE: A quick summary of Https Nginx -->

# NginxとLet's encryptでリバースプロキシを実現

[公式wikiのページ ](https://docs.requarks.io/wiki/admin-guide/setup-nginx)にもnginxをリバースプロキシとしてhttpsでウェブに公開する手順が載っているが、大分ハマってしまったので、ここに手順を残す

## SSL証明書の取得（Let's encrypt）

[Let's encrypt](https://letsencrypt.org/)は最近ワイルドカード証明書を発行してくれるようになりました。
参考1: [ACME v2 and Wildcard Certificate Support is Live](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579)
参考2[ワイルカードSSLサーバ証明書とは](https://www.websecurity.symantec.com/ja/jp/theme/ssl-wildcard)

