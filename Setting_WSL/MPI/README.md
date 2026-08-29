# MPI環境の構築（Intel oneAPI）

最終更新: 2026-08 ／ 対象: Intel oneAPI 2025系以降、Ubuntu 22.04 / 24.04 / 26.04

並列計算を行うためのMPI環境を、Intel oneAPI Toolkitを用いて構築します。LAMMPSやOpenMXをビルドする際の前提となる環境です。

研究室の計算サーバにはすでに導入済みのため、サーバ上で作業する場合は「環境設定を反映する」まで読み飛ばして構いません。

---

## 1. 必要なパッケージを準備する

```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential cmake -y
```

---

## 2. Intel oneAPI Toolkit を入手してインストールする

必要なのは以下の2つです。MPI（Intel MPI）とコンパイラは Base Toolkit と HPC Toolkit に分かれて含まれています。

- Intel oneAPI Base Toolkit
- Intel oneAPI HPC Toolkit

[Intel oneAPI Toolkits のダウンロードページ](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html)にアクセスし、それぞれ以下を選択してインストーラを入手します。

- Operating System: **Linux**
- Distribution: **Offline**（オンライン版よりも途中で失敗しにくく、おすすめです）

> **注意**: WSL上のUbuntuで使う場合も、選ぶのは Windows 版ではなく **Linux 版**です。WSLのUbuntuは独立したLinux環境として動作します。

ダウンロードページで得られるURLは版ごとに異なり、しばらくすると失効します。そのため、ここには具体的なURLを記載していません。ページ上で最新版のリンクを確認し、`wget` の引数に貼り付けるか、ブラウザでダウンロードしてWSL側へコピーしてください。

過去のバージョンが必要な場合は、[インストールガイド](https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/current/overview.html)から該当バージョンのページを辿ると入手できることがあります。

入手したインストーラは、以下のように実行します（ファイル名は入手した版に読み替えてください）。

```
sudo sh ./intel-oneapi-base-toolkit-XXXX.sh
sudo sh ./intel-oneapi-hpc-toolkit-XXXX.sh
```

既定では `/opt/intel/oneapi` 以下にインストールされます。

---

## 3. 環境設定を反映する（作業のたびに実施）

oneAPIのコンパイラやMPIを使うには、シェルごとに環境変数を読み込む必要があります。

```
source /opt/intel/oneapi/setvars.sh
```

正しく読み込めたか確認します。バージョンなどが表示されればOKです。

```
icx -v          # Cコンパイラ
mpiifx -v       # MPI対応Fortranコンパイラ
mpirun --version
```

> **補足**: oneAPI 2024以降、従来の `icc` / `ifort`（Classicコンパイラ）は廃止され、`icx` / `ifx` に置き換わりました。MPI用のラッパも `mpiicc` / `mpiifort` から `mpiicx` / `mpiifx` に変わっています。古い手順書のコマンドが通らない場合は、この違いを確認してください。
>
> ただしスーパーコンピュータなど、サイト側が用意した環境ではClassicコンパイラが引き続き提供されている場合があります。阪大SQUIDがその例です（[OpenMXのSQUIDでのコンパイル](../../OpenMX/OpenMX_installation2/README.md)を参照）。

### （参考）シェル起動時に自動で読み込む場合

毎回コマンドを打つ手間を省きたい場合は、`~/.bashrc` の末尾に追記します。

```
cd
vim .bashrc
```

最終行に以下を追加して保存します。

```
source /opt/intel/oneapi/setvars.sh
```

反映させます。

```
source ~/.bashrc
```

> **注意**: この方法はすべてのシェル起動時にoneAPIの環境変数（`PATH`、`LD_LIBRARY_PATH`、`CPATH` など）を書き換えます。システムのgccやPython環境（condaなど）と衝突して、oneAPIと無関係な作業で予期しない挙動が起きることがあります。複数の計算環境を使い分ける場合は、自動読み込みにせず、必要なときだけ手動で `source` するほうが安全です。

---

## 関連ページ

- [LAMMPSのインストール（基本版）](../../LAMMPS/LAMMPS_installation/README.md)
- [LAMMPS環境構築ガイド（統合版）](../../LAMMPS/LAMMPS_installation2/README.md)
- [OpenMXのインストール](../../OpenMX/OpenMX_installation/README.md)
