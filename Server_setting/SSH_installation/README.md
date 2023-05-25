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
- X11Forwardingを許可
```
X11Forwarding yes
```
- その他詳細は秘密。
