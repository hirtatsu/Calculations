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
- 例として、Ubuntuのイメージファイルを元にして、aptのupdateしたのちにxterm, vim, wget, sudoをインストールする内容を記入して保存する。
```
FROM ubuntu
RUN apt update
RUN apt install xterm vim wget sudo -y
```
### Dockerfileを使ったイメージ作成
- Dockerfileを使ってイメージを作成する
```
docker build . -t イメージ名
```
### Dockerのコンテナ作成・起動
- 作成したイメージを使ってコンテナを作成・起動する
```
docker run -dit --name コンテナ名 -e DISPLAY=$DISPLAY イメージ名 /bin/bash
```
- コンテナが無事起動していることを確認する
```
docker ps
```

- コンテナに入る
```
docker exec -it コンテナ名 /bin/bash
```

これにて、Docker上にUbuntu環境が構築できました。
