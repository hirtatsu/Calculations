## Docker上へのLinux環境構築１
### Ubuntuコンテナ作成と起動
ここでは、
- コンテナ名を「ubuntu-test」
- オプションに「-it」(-d: バックグラウンド実行, -i: コンテナにキーボードを繋ぐ, -t: 特殊きーを使用可能にする)を指定
- イメージ名は「ubuntu」
- 引数に「/bin/bash」(シェルコマンド)を指定
```
docker run --name ubuntu-test -dit ubuntu /bin/bash
```

### Ubuntu@Dockerに入る
- コンテナ名を「ubuntu-test」
- オプションに「-it」(-i: コンテナにキーボードを繋ぐ, -t: 特殊きーを使用可能にする)を指定
- 引数に「/bin/bash」(シェルコマンド)を指定
```
docker exec -it ubuntu-test /bin/bash
```

- Ubuntu@Dockerに入れたことを確認する
```
ls # rootディレクトリの内容が表示される
cat /etc/issue # Ubuntuのバージョンが表示される
```

- 現在ログインしているユーザ名を確認する(rootになっているはず)
```
whoami
```

### Ubuntu@Dockerを終了する
- Ubuntu@Dockerのコマンドラインで終了する。
```
exit
```
- コンテナはバックグラウンド実行のままなので、コンテナを停止したい場合は以下。
```
docker stop ubuntu-test
```

## Ubuntuコンテナにおける環境構築
### ユーザの作成と権限付与
- 今一度、Ubuntu@Dockerに入る
```
docker start ubuntu-test
docker exec -it ubuntu-test /bin/bash
```

- sudoパッケージのインストール
```
apt update
apt install sudo
```

- ユーザの作成
```
sudo adduser ユーザ名
```
まずパスワードを聞かれるので入力。その後にいろいろ聞かれるけど、何も入力せずにEnter連打して完了。

- 作成したユーザにsudo権限を付与
```
sudo usermod -aG sudo ユーザ名
```

- 作成したユーザでログインしなおす
```
su ユーザ名
```

- ログインしているユーザ名が変更されていることを確認する
```
whoami
```

- 次回以降、ユーザ指定してUbuntuコンテナに入ることができる
```
docker exec -it --user=ユーザ名 ubuntu-test /bin/bash
cd ~/ # ホームディレクトリに移動する
```

### 基本的なパッケージをインストールする
- 準備
```
sudo apt update
```
- インストール
```
sudo apt install vim wget build-essential -y
```

