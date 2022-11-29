## いろいろセッティング
### ユーザ管理
- これで自動的に/home/ユーザディレクトリを作ってくれる
```
sudo adduser ユーザ名
```

- (必要あれば) ユーザにsudo権限を付与する
```
sudo usermod -aG sudo ユーザ名
```

- 不要なユーザを削除する
```
sudo userdel -r ユーザ名
```
- ユーザの一覧を確認する
```
cat /etc/passwd
```

- 私は誰？
```
whoami
```

- 現在ログインしているユーザの確認
```
who # または
users
```

