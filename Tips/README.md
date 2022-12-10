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

### ストレージの増設とマウントと/homeディレクトリに設定
- 現状確認
```
df -h
```

### カレントディレクトリ以下の、サブディレクトリ内にあるJPGファイルを検索して、/tempにコピーする
```
find . -name "*.jpg" | xargs -I {} cp {} ./temp
```

### 特定のディレクトリやファイルを除いて、dir1以下をサブディレクトリ含めてdir2に一括してコピーする
- まずは実行せずに確認だけ
```
rsync -ahvn ./dir1/ ./dir2/ --exclude 'XXX' --exclude 'YYY' --exclude 'ZZZ'
```
- 問題なければ実行する (オプションのnを消す)
```
rsync -ahv ./dir1/ ./dir2/ --exclude 'XXX' --exclude 'YYY' --exclude 'ZZZ'
```

### find + sed で一斉置換
```
# ファイル名のパターンで対象ファイルを検索し、中身の文字列hogeをHOGEに買い替える
find . -type f -name "*.txt" -print0 | xargs -0 sed -i -e "s/hoge/HOGE/"
```
