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
以下のように

![Localhost 8080](/uploads/img/localhost-8080.png "Localhost 8080")

## githubのrepoにpushできない

以下の手順を行う  
なお、ssh用の鍵を作成済み、*Wiki.js* に設定済み、かつ*Github*にpublic keyを登録済みだと仮定する

1. `/home/username/.ssh/config`の設定  
```
Host github.com
HostName github.com
User git
IdentityFile /home/{username}/.ssh/{privatekeyname}
```

2. `git remote -v`で設定を確認  
```console
$ cd /path/to/wiki/
$ cd repo
$ git remote -v
origin	https://github.com/{github_username}/{reponame} (fetch)
origin	https://github.com/{github_username}/{reponame} (push)
```  
このようにpushに*https*を使うようになっていたら、
```console
$ git remote set-url origin git@github.com:{github_username}/{reponame}.git
$ git remote -v
origin	git@github.com:{github_username}/{reponame}.git (fetch)
origin	git@github.com:{github_username}/{reponame}.git (push)
```  
このようにして*ssh*を使うように変更する

## 再起動のたびにremote originが勝手に切り替わる

sshでgitを使うように設定している場合の話です  
（ほとんどそうだと思うけど）

- 正しい状態（*ssh*を使うよう設定されている）

```console
$ git --git-dir=/path/to/wiki/repo/.git remote -v
origin	git@github.com:{username}/{reponame}.git (fetch)
origin	git@github.com:{username}/{reponame}.git (push)
```

- 異常時（*https*を使うように設定が変わっている）

```console
$ git --git-dir=/path/to/wiki/repo/.git remote -v
origin	https://github.com/{username}/{reponame} (fetch)
origin	https://github.com/{username}/{reponame} (push)
```

このように、再起動（`pm2 reload wiki`した場合も含めて）したときに、勝手に切り替わってしまう症状が発生した
こうなる理由として、`config.yml`がうまく設定されていないことが考えられる

```yaml
git:
  url: 'https://github.com/{username}/{reponame}'
  branch: master
```

こうなっていると、`node wiki configure`で設定したどうかに関わらず、`node wiki restart`や`pm2 reload wiki`するたびにremote originが*https*のものになってしまう  
以下のように書き換えることで解決する

```yaml
git:
  url: 'git@github.com:{username}/{reponame}'
  branch: master
```

これで`pm2 reload wiki`、`node wiki restart`しても *ssh* をremote originにする設定が書き換わらなかった
