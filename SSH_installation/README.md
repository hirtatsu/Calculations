## SSH接続のための初期設定
### sshのインストール
```
sudo apt update
sudo apt install ssh
systemctl start sshd
```

### sshd_configファイルの編集
- 以下で編集モードに入る
```
sudo vim /etc/ssh/sshd_config
```
- 変更する場所はこちら
