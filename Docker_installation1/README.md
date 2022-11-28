# Linux上へのDockerのインストール方法
いろいろなソフトウェアを同じ環境下で開発・計算していく場合、共有される実行環境やライブラリの変更によって、あるソフトウェアが動かなくなることがよくあります。
そこで、環境を隔離することができるのがDockerです。

- Linux OS上に、コンテナと呼ばれる複数の隔離されたOS(っぽいもの)を、それぞれが独立した状態で構築することができます。
- 構築した環境をイメージと呼ばれるファイルに換えて、容易に複製したり移行したり配布したりできます。
- 仮想化するよりも軽いのが大きなメリットです。

公式マニュアルは[こちら](https://docs.docker.com/engine/install/ubuntu/)。

## Docker Engine on Ubuntuをインストールする

### 公式マニュアルは[こちら](https://docs.docker.com/engine/install/ubuntu/#set-up-the-repository)。

- 旧バージョンをアンインストール
```
sudo apt remove docker docker-engine docker.io containerd runc
```

- インストールする
まずリポジトリを設定する
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```
- Docker EngineのGPGキーを設定する
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
- 以下のコマンドでリポジトリを設定する
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- Docker Engineをインストールする
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

- 管理者以外も利用できるようにする。以下を入力したのち、いったんログアウトして再ログインする。
```
sudo usermod -aG docker $USER
```
- Docker Engineを起動、終了する。
```
sudo systemctl start docker # 起動
sudo systemctl stop docker # 終了
```

- ※WSL上の場合は以下を実行してDocker Daemonを起動する
```
sudo service docker start
sudo service docker stop
```

- 無事インストールされているか確認する
```
sudo docker run hello-world
```
