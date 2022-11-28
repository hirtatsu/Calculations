## Dockerを使ったUbuntu環境構築
- Dockerfileを使うと、書かれている内容の通りのDockerイメージを自動で作成してくれます。

### Dockerfileの作成
- ファイル名「Dockerfile」を作成
```
touch Dockerfile
```
- Dockerfileの編集
```
vim Dockerfile
```
- Ubuntuのイメージファイルを元にする。
- aptのupdateしたのちにxterm, vim, wget, sudoをインストールする
- 新規ユーザ(user)を作成する
- userのパスワードを8685とする
- userにsudo権限を付与する
```
FROM ubuntu
RUN apt update
RUN apt install xterm vim wget sudo -y
RUN sudo useradd --create-home user
RUN echo "user:8685"|chpasswd 
RUN sudo usermod -aG sudo user
```
### Dockerfileを使ったイメージ作成
- Dockerfileを使ってイメージを作成する。ここでは作成するイメージ名をimage-ubuntuとする。
```
docker build . -t image-ubuntu
```
### Dockerのコンテナ作成・起動
- 作成したイメージを使ってコンテナを作成・起動する。ここでは、
  - コンテナ名を「test-ubuntu」
  - オプションに「-dit」(-d: バックグラウンド実行, -i: コンテナにキーボードを繋ぐ, -t: 特殊きーを使用可能にする)を指定
  - -e DISPLAY=$DISPLAYにて、DISPLAYの環境変数を受け渡す
  - イメージ名は「image-ubuntu」
  - 引数に「/bin/bash」(シェルコマンド)を指定
```
docker run -dit --name test-ubuntu -e DISPLAY=$DISPLAY --user=user image-ubuntu /bin/bash
```
- コンテナが無事起動していることを確認する
```
docker ps
```

- コンテナに入る
  - コンテナ名を「ubuntu-test」
  - オプションに「-it」(-i: コンテナにキーボードを繋ぐ, -t: 特殊きーを使用可能にする)を指定
  - 引数に「/bin/bash」(シェルコマンド)を指定
```
docker exec -it test-ubuntu /bin/bash
```


- Ubuntu@Dockerに入れたことを確認する
```
ls # rootディレクトリの内容が表示される
cat /etc/issue # Ubuntuのバージョンが表示される
```

- 現在ログインしているユーザ名を確認する(userになっているはず)
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


これにて、Docker上にUbuntu環境が構築できました。
