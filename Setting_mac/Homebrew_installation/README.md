# Homebrewのインストール（Mac）

最終更新: 2026-08 ／ 対象: macOS（Apple Silicon / Intel）

Homebrewは、macOS向けのパッケージ管理ソフトウェアです。Linuxでいう `apt` や `yum` に相当し、コマンド1つで各種ソフトウェアを導入・更新できます。

---

## インストール

1. アプリケーションから**ターミナル**を起動します
2. [Homebrew公式ページ](https://brew.sh/ja/)にアクセスします
3. ページに記載されているインストールコマンドをターミナルに貼り付けて実行します

コマンドは以下の形です（公式ページで最新のものを確認してください）。

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- パスワードを聞かれたら、Macのログインパスワードを入力します（入力しても画面には表示されません）
- Command Line Tools が未導入の場合、あわせてインストールされます

---

## PATHを通す（Apple Siliconでは必須）

Apple Silicon搭載Macでは、Homebrewは `/opt/homebrew` にインストールされます。**この場所は初期状態でPATHに含まれていないため、インストール直後は `brew` コマンドが見つかりません。**

インストーラは完了時に **Next steps** として実行すべきコマンドを表示します。その指示に従ってください。表示を見逃した場合は、以下を実行します。

```
echo >> ~/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

macOSの既定シェルはzshのため、設定は `~/.zprofile` に書き込みます。

> **Intel Macの場合**: インストール先が `/usr/local` となり、こちらは既定でPATHに含まれているため、この操作は不要です。
>
> なお、macOS Tahoe（26）がIntel Macに対応する最後の世代とされています。以降の記述はApple Siliconを前提とします。

---

## 動作確認

以下でバージョンが表示されればインストール完了です。

```
brew --version
```

`command not found` と表示される場合は、前項のPATH設定が済んでいません。

インストール環境に問題がないかは以下で確認できます。

```
brew doctor
```

---

## よく使うコマンド

| 操作 | コマンド |
|---|---|
| パッケージリストの更新 | `brew update` |
| インストール済みパッケージの更新 | `brew upgrade` |
| パッケージの検索 | `brew search パッケージ名` |
| パッケージの情報を表示 | `brew info パッケージ名` |
| パッケージのインストール | `brew install パッケージ名` |
| パッケージのアンインストール | `brew uninstall パッケージ名` |
| インストール済み一覧 | `brew list` |
| 不要になった依存パッケージの削除 | `brew autoremove` |
| キャッシュの削除 | `brew cleanup` |

GUIアプリケーション（`.app`）を導入する場合は `--cask` を付けます。

```
brew install --cask アプリ名
```

---

## 関連ページ

- [X Window Systemのセットアップ（Mac）](../Xwindowsystem_mac_installation/README.md) — XQuartzの導入に使用します
- [Pythonの環境構築（Mac）](../Python_mac_installation/README.md)
