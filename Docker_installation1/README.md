## Linux上へのDockerのインストール方法
いろいろなソフトウェアを同じ環境下で開発・計算していく場合、共有される実行環境やライブラリの変更によって、あるソフトウェアが動かなくなることがよくあります。
そこで、環境を隔離することができるのがDockerです。

- Linux OS上に、コンテナと呼ばれる複数の隔離されたOS(っぽいもの)を、それぞれが独立した状態で構築することができます。
- 構築した環境をイメージと呼ばれるファイルに換えて、容易に複製したり移行したり配布したりできます。
- 仮想化するよりも軽いのが大きなメリットです。

### 環境
Ubuntu on WSL2

### Dockerの環境構築
- Ubuntu on WSL上で必要なパッケージをインストールする
```
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common  -y
```
- PGPキーの追加
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

- フィンガープリントの確認
```
sudo apt-key fingerprint 0EBFCD88 
```

- リポジトリに追加して更新
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
```

- Docker本体のインストール
```
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

- 管理者以外も利用できるようにする
```
sudo usermod -aG docker $USER
```
そのあと一旦ログアウトしてから再びログインしなおす。

- インストールできたか確認
```
docker --version
```
と入力して、Versionに関する表示が出たら成功。
