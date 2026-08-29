# Linux作業Tips（備忘録）

最終更新: 2026-08

計算サーバやWSL上のUbuntuで折に触れて使う操作をまとめた備忘録。体系的な入門ではなく、必要になったときに引くための断片集。

---

## ユーザ管理

主に計算サーバの管理者作業。

- ユーザを追加する。`/home/ユーザ名` は自動的に作成される。

```
sudo adduser ユーザ名
```

- （必要あれば）ユーザに sudo 権限を付与する

```
sudo usermod -aG sudo ユーザ名
```

- 不要なユーザをホームディレクトリごと削除する

```
sudo userdel -r ユーザ名
```

- 一般ユーザの一覧を確認する。`cat /etc/passwd` でも見られるが、システムユーザが大量に混ざるため UID で絞ったほうが読みやすい。

```
getent passwd {1000..60000}
```

- 自分が誰か / 現在ログイン中のユーザを確認する

```
whoami   # 自分
who      # ログイン中のユーザ（users でも可）
```

---

## ストレージの増設とマウント

新しいディスクを追加し、常時マウントして使えるようにするまでの一連の流れ。

> **注意**: `/etc/fstab` の記述を誤ると、次回起動時にシステムが起動しなくなることがある。後述の `nofail` オプションと、再起動前の `mount -a` による検証を必ず行うこと。

### 1. 現状とデバイス名の確認

```
df -h        # マウント済み領域の使用状況
lsblk -f     # 接続されているディスク・パーティション・ファイルシステム
```

`lsblk` の出力から、増設したディスクのデバイス名（`/dev/sdb` など）を特定する。以下では `/dev/sdX` と表記する。**デバイス名を取り違えると既存のデータを破壊するので、容量と型番で必ず確認すること。**

### 2. パーティションの作成（GPT）

```
sudo parted /dev/sdX
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) print
(parted) quit
```

### 3. ファイルシステムの作成

```
sudo mkfs.ext4 -L data /dev/sdX1
```

`-L` はラベル。省略しても構わない。

### 4. マウントポイントの作成と手動マウントの確認

```
sudo mkdir -p /data
sudo mount /dev/sdX1 /data
df -h /data
```

### 5. 起動時に自動マウントさせる

デバイス名（`/dev/sdX1`）は接続順で変わることがあるため、UUID で指定する。

```
sudo blkid /dev/sdX1
```

表示された UUID を使い、`/etc/fstab` の末尾に1行追記する。

```
sudo vim /etc/fstab
```

```
UUID=ここにUUID  /data  ext4  defaults,nofail  0  2
```

`nofail` を付けておくと、ディスクが見つからない場合でもシステムは起動する。

### 6. 検証（再起動前に必ず実施）

```
sudo umount /data
sudo mount -a
df -h /data
```

エラーが出ず `/data` がマウントされていれば `/etc/fstab` の記述は正しい。ここでエラーが出たまま再起動しないこと。

### 増設したディスクを /home に割り当てる場合

既存のホームディレクトリを新しいディスクへ移す手順。**作業中は対象ユーザが誰もログインしていない状態で行う。**

```
# 新ディスクを一時的な場所にマウントして中身を複製する
sudo mkdir -p /mnt/newhome
sudo mount /dev/sdX1 /mnt/newhome
sudo rsync -aHAX --info=progress2 /home/ /mnt/newhome/
```

`-aHAX` はパーミッション・所有者・ハードリンク・ACL・拡張属性を保持する指定。ホームディレクトリの移行では必須。

複製後、内容を突き合わせて確認する。

```
sudo diff -r /home /mnt/newhome
```

問題なければ旧 `/home` を退避し、マウント先を差し替える。

```
sudo umount /mnt/newhome
sudo mv /home /home.old
sudo mkdir /home
```

`/etc/fstab` のマウント先を `/data` から `/home` に書き換えてから、

```
sudo mount -a
ls /home
```

正常にログインできることを確認できてから `/home.old` を削除する。**確認前に消さないこと。**

### 容量を食っている場所を調べる

```
du -sh /path/to/dir          # 指定ディレクトリの合計
du -h --max-depth=1 /home | sort -h   # 直下のディレクトリを大きい順に
```

対話的に掘り下げたい場合は `ncdu`（`sudo apt install ncdu`）が速い。

---

## ファイル操作

### サブディレクトリ内のJPGファイルを検索して一括コピーする

カレントディレクトリ以下から `*.jpg` を探し、`./temp` にコピーする。

```
mkdir -p ./temp
find . -name "*.jpg" -print0 | xargs -0 -I{} cp {} ./temp/
```

`-print0` と `-0` の組み合わせにより、空白を含むファイル名でも壊れない。

### 特定のディレクトリやファイルを除いてディレクトリごとコピーする

`dir1` 以下をサブディレクトリ含めて `dir2` へコピーする。まず `-n`（dry run）で確認する。

```
rsync -ahvn ./dir1/ ./dir2/ --exclude 'XXX' --exclude 'YYY' --exclude 'ZZZ'
```

問題なければ `n` を外して実行する。

```
rsync -ahv ./dir1/ ./dir2/ --exclude 'XXX' --exclude 'YYY' --exclude 'ZZZ'
```

コピー元の末尾の `/` の有無で挙動が変わる（`dir1/` は中身を、`dir1` はディレクトリごとコピーする）。

### find + sed で文字列を一斉置換する

対象ファイルをファイル名のパターンで検索し、中身の文字列 `hoge` を `HOGE` に置き換える。

```
# まず置換対象を確認する
find . -type f -name "*.txt" -print0 | xargs -0 grep -l "hoge"

# 問題なければ実行する
find . -type f -name "*.txt" -print0 | xargs -0 sed -i -e "s/hoge/HOGE/g"
```

`sed -i` は元ファイルを直接書き換えるため、実行前に対象を確認すること。末尾の `g` を付けないと各行の最初の1件しか置換されない。
