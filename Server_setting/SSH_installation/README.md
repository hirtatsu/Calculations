## SSH接続のための初期設定
### sshのインストール
```
sudo apt update
sudo apt install openssh-server
```

### 起動状態の確認
```
sudo systemctl status ssh
```
`active`と表示されていればOK。

|操作	| コマンド|
|---|---|
|SSHサーバーの停止	|sudo systemctl stop ssh|
|SSHサーバーの起動	|sudo systemctl start ssh|
|SSHサーバーの再起動	|sudo systemctl restart ssh|
|設定のリロード	|sudo systemctl reload ssh|
|自動起動の有効化	|sudo systemctl enable ssh|
|自動起動の無効化	|sudo systemctl disable ssh|
