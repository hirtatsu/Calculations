# Matlantis環境構築

最終更新: 2026-08 ／ 対象: Matlantis（クラウド環境）

Matlantisは、汎用ニューラルネットワークポテンシャル **PFP（PreFerred Potential）** を用いた原子シミュレーションのクラウドサービスです。第一原理計算に近い精度を保ちながら、桁違いに高速な構造最適化やMD計算を行えます。

> **利用にはライセンス契約が必要です。** 株式会社Preferred Computational Chemistryが提供する商用サービスで、契約者はブラウザ上のJupyterLab環境から利用します。公式情報は[Matlantis公式サイト](https://matlantis.com/ja/)を参照してください。

以下は、Matlantisのターミナル上で行う環境構築の手順です。

---

## 1. シェルの初期設定

Matlantisのターミナルを開き、`.bashrc` を編集します。

```
cd
vim .bashrc
```

末尾に以下を追記します。

```
use_venv python313    # Matlantisが提供する仮想環境を読み込む（抜けるには deactivate_venv）
alias ls='ls --color=auto -ltr'    # lsの表示を好みに変更
```

反映させます。

```
source .bashrc
```

> `use_venv` はMatlantis環境が提供する仮想環境の切り替えコマンドです（pyenvやcondaとは別のものです）。指定できる環境名は提供状況によって変わるため、利用可能なバージョンは実際の環境で確認してください。

---

## 2. PFPの利用環境を構築する

PFPを呼び出すためのクライアントライブラリを導入します。原子シミュレーション用のライブラリ **ASE**（Atomic Simulation Environment）も同時にインストールされます。

```
pip install --upgrade pip
pip install pfp-api-client
```

---

## 3. SSH接続の設定

ローカル環境からMatlantisへSSH接続する場合の設定です。公式ドキュメントに従ってください。

- [SSH接続 設定方法（Matlantis公式）](https://matlantis.zendesk.com/hc/ja/articles/40524152374169-SSH%E6%8E%A5%E7%B6%9A-%E8%A8%AD%E5%AE%9A%E6%96%B9%E6%B3%95)

---

## 4. Atomskのインストール

計算モデルを作成するためのツールです。使い方は[Atomskのインストールと使い方](../LAMMPS/Atomsk_installation/README.md)を参照してください。

> **注意**: Matlantisの環境では `dpkg` によるDebianパッケージのインストールが行えないため、**静的リンクされたバイナリ版（tar.gz）を展開して使います。** Ubuntu上での導入手順（deb版）とは異なります。

### 手順

[Atomskのダウンロードページ](https://atomsk.univ-lille.fr/dl.php)から「Linux amd64 (64 bits)」版を入手します。

作業用ディレクトリを作成して移動します。

```
mkdir tool
cd tool
```

ダウンロードしたファイルを、このディレクトリにアップロード（ドラッグ＆ドロップ）します。

展開してインストールスクリプトを実行します。

```
tar -xzvf atomsk_bX.XX.X_Linux-amd64.tar.gz    # 入手したファイル名に読み替え
cd atomsk_bX.XX.X_Linux-amd64
sh install.sh
```

PATHが追加されるので反映させます。

```
source ~/.bashrc
```

動作確認します。

```
atomsk --create fcc 4.02 Al aluminium.xsf
```

XSFファイルが生成されれば成功です。

---

## 5. LAMMPSの利用（PFPを力場としたMD計算）

PFPをLAMMPSの力場として使うための連携パッケージが提供されています。導入手順は公式ドキュメントに従ってください。

- [LAMMPS連携パッケージ（matlantis-lammps）のインストール方法（Matlantis公式）](https://matlantis.zendesk.com/hc/ja/articles/33994518415257-LAMMPS%E9%80%A3%E6%90%BA%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8-matlantis-lammps-%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E6%96%B9%E6%B3%95)

インストールが完了すると、LAMMPSの実行ファイル（`lmp_serial`）が仮想環境下に配置され、PATHが通った状態になります。

通常のLAMMPSとの違いや、入力ファイルの書き方については公式ドキュメントを参照してください。LAMMPS自体の基本的な使い方は[LAMMPSのインストール（基本版）](../LAMMPS/LAMMPS_installation/README.md)にまとめています。

---

## 関連ページ

- [Atomskのインストールと使い方](../LAMMPS/Atomsk_installation/README.md)
- [LAMMPSのインストール（基本版）](../LAMMPS/LAMMPS_installation/README.md)
- [OVITOの使い方メモ](../LAMMPS/OVITO_tips/README.md) — 計算結果の可視化
