# MateriApps LIVE!のインストール

最終更新: 2026-08 ／ 対象: MateriApps LIVE! 5.0系

計算物質科学のアプリケーションが一通り導入済みのLinux環境を、手持ちのPC上で動かせるようにしたものです。東京大学物性研究所ほかが提供しています。

**計算環境の構築で最初の難関になるソフトウェアのコンパイルを行わずに、すぐ計算を試せます。** これから計算研究を始める方が、まず触ってみるのに適しています。

- 公式サイト: [https://cmsi.github.io/MateriAppsLive/](https://cmsi.github.io/MateriAppsLive/)
- ダウンロード: [Wiki内のダウンロードページ](https://github.com/cmsi/MateriAppsLive/wiki/download)

含まれる主なアプリケーションは、OpenMX、Quantum ESPRESSO、ABINIT、LAMMPS、Gromacs、ALAMODE、HPhi など。可視化ツールとして OVITO、VESTA、gnuplot、ParaView なども入っています。一覧は[こちら](https://github.com/cmsi/MateriAppsLive/wiki/ApplicationsAndTools)。

---

## 版の選び方

MateriApps LIVE! には **VirtualBox版** と **Docker版** の2種類があります。お使いの環境に応じて選んでください。

| 環境 | 推奨 | 入手するファイル |
|---|---|---|
| Windows（通常） | VirtualBox版 | `MateriAppsLive-5.0-amd64.ova` |
| Windows（WSL2を使用中） | Docker版 | — |
| Mac（Intel） | VirtualBox版 または Docker版 | `MateriAppsLive-5.0-amd64.ova` |
| Mac（Apple Silicon） | VirtualBox版 または Docker版 | `MateriAppsLive-5.0-arm64.ova` |

> ~~M1 Mac上ではVirtualBoxが動かないので、別途用意されているDocker版を使うとよい~~
>
> **この記述は古くなりました（2026-08時点）。** バージョン5.0以降、**Apple Silicon向けのVirtualBox版（`arm64.ova`）が提供されています。** Apple Silicon搭載Macでも、VirtualBox版・Docker版のどちらでも利用できます。

初めて使う場合は、GUIのデスクトップ環境がそのまま起動するVirtualBox版のほうが分かりやすいと思います。

---

## VirtualBox版の導入

### 1. VirtualBoxをインストールする

[VirtualBox公式サイト](https://www.virtualbox.org/wiki/Downloads)から、お使いのOS向けのインストーラをダウンロードして実行します。**バージョン7.0以降**が必要です。

### 2. ディスクイメージをダウンロードする

[ダウンロードページ](https://github.com/cmsi/MateriAppsLive/wiki/download)から、上の表で示した `.ova` ファイルを入手します。

> **ファイルサイズは約3 GBあります。** 回線状況によっては時間がかかるので、余裕のあるときにダウンロードしてください。

### 3. インポートする

- `.ova` ファイルをダブルクリックします
- VirtualBoxが起動してインポート画面が表示されるので、**インポート** を押します
- 数分待つと完了し、VirtualBoxマネージャーの一覧に表示されます

### 4. 起動してログインする

- VirtualBoxマネージャーで `MateriAppsLive-5.0-...` を選択し、**起動** を押します
- ログイン画面が表示されるまで待ちます

ログイン情報は以下のとおりです。

| 項目 | 値 |
|---|---|
| ユーザ名 | `user` |
| パスワード | `live` |

デスクトップ画面が表示されれば成功です。ターミナルは **スタートメニュー → System Tools → LXTerminal** から開けます。

---

## Docker版の導入

WSL2をすでに使っている場合や、仮想マシンを起動せずに使いたい場合に向いています。

導入手順は公式Wikiに詳しくまとまっているので、そちらに従ってください。

- [MateriApps LIVE! Docker版の起動方法](https://github.com/cmsi/MateriAppsLive/wiki/GettingStartedDocker)

概要は以下のとおりです。

1. [Docker Desktop](https://www.docker.com/products/docker-desktop/) をインストールする（**Macの場合、Intel用とApple Silicon用があるので、使用しているMacに合うものを選んでください**）
2. Macの場合は[XQuartz](https://www.xquartz.org/)をインストールする（GUIアプリの表示に必要）
3. 起動スクリプト（`malive`）を導入して実行する

> **Macで詰まりやすい点**: XQuartzをインストールした後、**XQuartzの環境設定 → セキュリティタブで「接続を認証」と「ネットワーク・クライアントからの接続を許可」の両方にチェックを入れる**必要があります。これを行わないとGUIが表示されません。設定後はXQuartzを再起動してください。
>
> XQuartzの導入手順は[X Window Systemのセットアップ（Mac）](../../Setting_mac/Xwindowsystem_mac_installation/README.md)も参照してください。

---

## 使い始めるにあたって

### 設定資料

キーボード配列の設定、ホストPCとのファイル共有など、最初に行っておくとよい設定が[setup.pdf](https://github.com/cmsi/malive-tutorial/raw/master/setup/setup.pdf)にまとまっています。**まずこれに目を通してください。**

### ホストPCとファイルをやり取りする（VirtualBox版）

計算結果を手元のPCに取り出したい場合は、共有フォルダを設定しておくと便利です。

1. 手元のPCに共有用のフォルダを作成しておきます
2. VirtualBoxマネージャーで `MateriAppsLive-*` を選択し、**設定** を開きます
3. **共有フォルダー** タブを開き、右側の「+」をクリックします
4. **フォルダーのパス** で手順1のフォルダを選択します
5. **自動マウント** にチェックを入れて **OK** を押します
6. 仮想マシンを再起動すると、`/media/sf_フォルダ名` に表示されます

### 各アプリケーションの使い方

MateriAppsのポータルサイトに、アプリケーションごとのレビュー記事とチュートリアルがあります。

- 例: [OpenMXのレビュー記事](https://ma.issp.u-tokyo.ac.jp/app-post-category/review?appid=594)

### パッケージ情報の更新

MateriAppsのDebianリポジトリは2025年2月にAWS S3へ移行しました。**古いバージョンのMateriApps LIVE!を使っていて、パッケージの更新ができない場合**は、以下でリポジトリ情報を更新します。

```
curl -L https://malive.s3.amazonaws.com/repos/setup.sh | sudo /bin/sh
```

バージョン5.0を新規に導入した場合は、この操作は不要です。

---

## 困ったときは

- [MateriApps LIVE! Wiki](https://github.com/cmsi/MateriAppsLive/wiki) — FAQあり
- [MateriApps LIVE! Forum](https://github.com/cmsi/MateriAppsLive-forum/issues) — 利用者フォーラム

---

## 次のステップ

MateriApps LIVE! で計算に慣れたら、自分の環境に計算ソフトウェアを導入してみてください。

- [WSL2のインストール](../../Setting_WSL/WSL2_installation/README.md)
- [OpenMXのインストール](../../OpenMX/OpenMX_installation/README.md)
- [LAMMPSのインストール（基本版）](../../LAMMPS/LAMMPS_installation/README.md)
