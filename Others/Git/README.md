# Gitの設定と基本操作

最終更新: 2026-08 ／ 対象: Ubuntu（WSL2 / ネイティブ）、macOS

ソースコードや文書の変更履歴を管理するためのツールです。GitHubと組み合わせて、複数の環境で作業を同期したり、変更を記録として残したりします。

前半は最初に一度だけ行う設定、後半は日常的に使う操作です。**操作を思い出したいときは[日常の操作](#日常の操作)から読んでください。**

---

## 初期設定（最初に一度だけ）

### インストール

```
sudo apt update
sudo apt install git
```

macOSではHomebrewから導入できます（Command Line Toolsに同梱されているものでも構いません）。

```
brew install git
```

### ユーザ情報などの設定

コミットの記録に使われる名前とメールアドレス、エディタ、既定のブランチ名を設定します。

```
git config --global user.name 'GitHubのアカウント名'
git config --global user.email 'GitHubに登録しているメールアドレス'
git config --global core.editor 'vim'
git config --global init.defaultBranch main
```

> `init.defaultBranch main` は重要です。これを設定しないと、Gitが作る既定のブランチ名は `master` になりますが、GitHub側の既定は `main` です。食い違ったままだと最初のpushで混乱します。

設定内容を確認します。

```
git config --list
```

### SSH鍵の作成とGitHubへの登録

鍵を作成します。

```
ssh-keygen -t ed25519 -C "GitHubに登録しているメールアドレス"
```

- 保存先を聞かれたらそのままEnter（`~/.ssh/id_ed25519` に作成されます）
- パスフレーズは空でも動作しますが、設定しておくほうが安全です

公開鍵の内容をコピーします。

| 環境 | コマンド |
|---|---|
| WSL | `cat ~/.ssh/id_ed25519.pub \| clip.exe` |
| macOS | `cat ~/.ssh/id_ed25519.pub \| pbcopy` |
| その他 | `cat ~/.ssh/id_ed25519.pub` を実行して手動でコピー |

GitHubの[SSH鍵の設定ページ](https://github.com/settings/keys)を開きます。

1. **New SSH key** をクリック
2. **Title** に分かりやすい名前を入力（例: 研究室サーバ、自分のノートPC など）
3. **Key** にコピーした公開鍵を貼り付ける
4. **Add SSH key** をクリック

### 接続の確認

```
ssh -T git@github.com
```

途中で確認を求められたら `yes` を入力します。`Hi ユーザ名! You've successfully authenticated...` と表示されれば成功です。

---

## 日常の操作

### 全体の流れ

```
[GitHub上のリポジトリ]
       │  clone / pull（取得）
       ▼
[手元の作業ディレクトリ]  ── 編集 ──▶  add（記録対象に指定）
       ▲                                    │
       │  push（送信）                       ▼
       └──────────────────────────  commit（変更を確定）
```

`add` → `commit` → `push` が基本の3手順です。

### リポジトリを手元に持ってくる

```
git clone git@github.com:ユーザ名/リポジトリ名.git
cd リポジトリ名
```

### 変更を記録して送る（最頻出）

```
git status                      # 何が変更されたかを確認
git add .                       # 変更をすべて記録対象にする
git commit -m "変更内容の説明"    # 変更を確定する
git push                        # GitHubに送信する
```

`git add .` はカレントディレクトリ以下すべてを対象にします。特定のファイルだけを対象にする場合は以下のようにします。

```
git add ファイル名
```

### 他の環境での変更を取り込む

作業を始める前に実行しておくと、競合を避けられます。

```
git pull
```

### 状態を確認する

| 操作 | コマンド |
|---|---|
| 変更されたファイルの一覧 | `git status` |
| 変更内容の差分を見る | `git diff` |
| `add` 済みの差分を見る | `git diff --staged` |
| 変更履歴を見る | `git log --oneline` |
| 履歴をグラフ表示 | `git log --oneline --graph --all` |
| リモートリポジトリの確認 | `git remote -v` |

---

## 既存のディレクトリをGitで管理する

すでに手元にあるディレクトリを、新たにGitの管理下に置く場合の手順です。

```
cd 管理したいディレクトリ
git init
```

GitHub側で空のリポジトリを作成しておき、そこに接続します。

```
git remote add origin git@github.com:ユーザ名/リポジトリ名.git
git add .
git commit -m "first commit"
git push -u origin main
```

`-u` は最初の一回だけ必要です。以降は `git push` だけで送信できます。

### 管理対象から除外する（.gitignore）

計算結果の大容量ファイルや一時ファイルは、リポジトリに含めないほうがよい場合があります。ディレクトリ直下に `.gitignore` というファイルを作り、除外したいパターンを記述します。

```
vim .gitignore
```

```
# 計算結果
*.cube
dump.*
log.lammps

# 一時ファイル
*.tmp
.DS_Store

# ディレクトリごと除外
work/
```

> **注意**: すでにコミット済みのファイルは `.gitignore` に書いても除外されません。`git rm --cached ファイル名` で管理対象から外してからコミットしてください。

---

## よくある操作

### 直前のコミットをやり直す

コミットメッセージを書き間違えた、ファイルを含め忘れた、という場合に使います。

```
git add 忘れていたファイル        # 必要なら
git commit --amend
```

> **注意**: すでに `push` 済みのコミットに対して行うと、履歴が食い違います。自分だけが使うリポジトリでなければ避けてください。

### `add` を取り消す

```
git restore --staged ファイル名
```

ファイルの内容は変更されず、記録対象から外れるだけです。

### 編集を破棄して最後のコミットの状態に戻す

```
git restore ファイル名
```

> **注意**: この操作で失われた変更は元に戻せません。実行前に `git diff` で内容を確認してください。

### pushが拒否された場合

```
! [rejected] main -> main (fetch first)
```

他の環境からの変更がGitHub側に入っている状態です。取り込んでから送り直します。

```
git pull --rebase
git push
```

---

## 関連ページ

- [計算サーバへの接続方法](../../Server_setting/Howtoaccess_server/README.md) — サーバへのSSH鍵の設定
