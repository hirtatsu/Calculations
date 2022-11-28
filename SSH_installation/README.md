## SSH接続のための初期設定
### sshのインストール
```
sudo apt update
sudo apt install ssh
systemctl start sshd
```

### sshd_configファイルの編集はこちら
- 以下で編集モードに入る
```
sudo vim /etc/ssh/sshd_config
```
- 詳細は秘密。
