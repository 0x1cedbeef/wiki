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

## githubのrepoにpushできない

以下の手順を行う  
なお、ssh用の鍵を**github**として作成済み、*Wiki.js* に設定済み、かつ*Github*にpublic keyを登録済みだと仮定する

1. `/home/username/.ssh/config`の設定  
```
Host github.com
HostName github.com
User git
IdentityFile /home/username/.ssh/github
```

2. `git remote -v`で設定を確認  
```bash
$ cd wiki/repo
$ git remote -v
origin	https://github.com/{github_username}/{reponame} (fetch)
origin	https://github.com/{github_username}/{reponame} (push)
```  
こうなっていたら、
```bash
$ git remote set-url origin git@github.com:{github_username}/{reponame}.git
$ git remote -v
origin	git@github.com:{github_username}/{reponame}.git (fetch)
origin	git@github.com:{github_username}/{reponame}.git (push)
```

