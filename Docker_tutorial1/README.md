## Dockerの基本的な使い方
### Docker Engineの起動・終了
- Linux上で起動、終了する。
```
sudo systemctl start docker # 起動
sudo systemctl stop docker # 終了
```

- WSL上のLinux上で起動、終了する
```
sudo service docker start # 起動
sudo service docker stop # 終了
```

### コンテナの一覧表示
- 動いているコンテナの一覧表示
```
docker ps
```
- 存在するコンテナの一覧表示
```
docker ps -a
```

### コンテナの作成・起動、削除の練習
- コンテナの作成＆起動 (このとき、コンテナの元となるイメージが自動的にダウンロードされる)
```
docker run --name test1 -d httpd # 練習としてApacheのイメージを作成して起動する
```
- コンテナが正常に作成・起動できたことの確認
```
docker ps
```

- コンテナの停止
```
docker stop test1
```

- コンテナの起動
```
docker start test1
```

- コンテナの削除
```
docker stop test1 # まず停止する
docker rm test1 # 次に削除する
```

- コンテナが正常に削除されたことの確認
```
docker ps -a
```

### イメージの確認・削除の練習
コンテナを作成すると、自動的にイメージがダウンロードされて保存されています。
コンテナを削除してもイメージは削除されないので、適宜イメージを削除しましょう。
ただし、既存のコンテナで使っているイメージは削除できません。

- 存在するイメージを確認する
```
docker image ls
```

- イメージを削除する
```
docker image rm httpd # REPOSITORY名がhttpdというイメージを削除する場合
```

- イメージを削除できたことを確認する
```
docker image ls
```
### aa
