# X Window Systemのセットアップ（Mac）

最終更新: 2026-08 ／ 対象: macOS（Apple Silicon / Intel）、XQuartz 2.8系

研究室の計算サーバ上で動作するGUIアプリケーションを、手元のMacの画面に表示できるようにします。

サーバ上のソフトウェア（可視化ツールなど）を、ファイルを転送せずにその場で使いたい場合に利用します。

> **WindowsのWSLをお使いの場合**: WSLにはWSLgが標準搭載されており、追加のXサーバは不要です。詳細は[WSL2のインストール](../../Setting_WSL/WSL2_installation/README.md)を参照してください。macOSにはこれに相当する仕組みがないため、以下のXQuartzの導入が必要です。

---

## 1. XQuartzのインストール

XQuartzは、macOS上でX Window Systemを動作させるためのソフトウェアです。

[公式サイト](https://www.xquartz.org/)から `.pkg` ファイルをダウンロードしてインストールするか、Homebrewを使います。

```
brew install --cask xquartz
```

（Homebrewの導入は[こちら](../Homebrew_installation/README.md)を参照してください。）

インストール後、**一度ログアウトして再度ログインしてください。** `DISPLAY` 環境変数を反映させるために必要です。

> 2026年8月時点の最新版は 2.8.6（2026年7月リリース）です。この版でApple Silicon環境における描画の不具合が修正されています。インストールにはmacOS 10.13以降が必要です。

---

## 2. 設定の確認

ターミナルを開き、`DISPLAY` 環境変数が設定されていることを確認します。

```
echo $DISPLAY
```

`/private/tmp/com.apple.launchd.XXXXXX/org.xquartz:0` のような文字列が表示されれば成功です。何も表示されない場合は、ログアウトと再ログインが済んでいるか確認してください。

---

## 3. サーバに接続して動作確認する

X転送を有効にしてSSH接続します。`-X` オプションを付けます。

```
ssh -X ユーザ名@ホスト名
```

`~/.ssh/config` に設定を書いている場合は、以下のように記述しておくと毎回オプションを付ける必要がなくなります。

```
Host server
  HostName xxx.xxx.xxx.xxx
  User sampleuser
  IdentityFile ~/.ssh/id_ed25519_server
  ForwardX11 yes
```

（SSH接続の設定は[計算サーバへの接続方法](../../Server_setting/Howtoaccess_server/README.md)を参照してください。）

接続先のサーバで以下を実行し、マウスの動きに連動する目玉が表示されれば成功です。

```
xeyes
```

<img width="133" alt="xeyes" src="https://user-images.githubusercontent.com/64639043/204118509-92d7c6c8-0a77-45ad-9989-8eff1024dccf.png">

`xeyes` が見つからない場合は、サーバ側にX11のサンプルアプリが入っていません。管理者に導入を依頼するか、以下で導入します。

```
sudo apt install x11-apps
```

---

## 補足

### 表示が遅い場合

X転送はネットワーク経由で描画情報をやり取りするため、回線状況によっては動作が重くなります。以下の方法があります。

- `ssh -Y`（信頼済みX転送）を使うと動作が軽くなる場合があります。ただし、接続先を信頼できる場合に限って使用してください
- 大きなデータを可視化する場合は、結果ファイルを手元に転送してローカルで開くほうが快適なことが多いです（[ファイルの転送](../../Server_setting/Howtoaccess_server/README.md)を参照）

### 高解像度ディスプレイでの表示

XQuartzはRetinaディスプレイの高解像度表示に対応していないため、X11アプリはピクセルを引き伸ばした状態で表示されます。

---

## 関連ページ

- [Homebrewのインストール](../Homebrew_installation/README.md)
- [計算サーバへの接続方法](../../Server_setting/Howtoaccess_server/README.md)
