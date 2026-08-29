# OpenMXのインストール

最終更新: 2026-08 ／ 対象: Ver 4.0 および Ver 3.9.9

第一原理計算ソフトウェア OpenMX を導入します。

研究室では Ver 3.9.9 で構築した環境が現役の端末もあるため、**Ver 4.0 と Ver 3.9.9 の両方**を記載しています。これから新規に構築する場合は Ver 4.0 を選んでください。

---

## 導入方法の選び方

| 方法 | 向いている場合 | 手間 |
|---|---|---|
| **Ver 4.0 / Debianパッケージ** | WSLやUbuntu上でとにかく計算を始めたい | 小 |
| **Ver 4.0 / ソースビルド** | スパコンで動かす、コードを改変する、コンパイラを選びたい | 大 |
| **Ver 3.9.9 / ソースビルド** | 既存の計算環境を再現する、過去の結果と条件を揃える | 大 |

Ver 4.0 は2026年4月26日にリリースされ、パッチ 4.0.1（2026年5月8日）が公開されています。CPU版に加えてGPU版も提供されています。Ver 3.9 と 3.8 も引き続き公開されています。

配布物はすべて[OpenMX公式のダウンロードページ](https://www.openmx-square.org/download.html)にあります。バージョンやファイル名は更新されるため、以下では具体的なURLを記載していません。ダウンロードページで最新の情報を確認してください。

---

## 0. 前提: コンパイラとMPI環境

ソースからビルドする場合、Intel oneAPI（コンパイラとIntel MPI、MKL）が必要です。導入手順は[MPI環境の構築（Intel oneAPI）](../../Setting_WSL/MPI/README.md)を参照してください。

研究室の計算サーバには導入済みのため、環境を読み込むだけで構いません。

```
source /opt/intel/oneapi/setvars.sh
icx -v
```

Debianパッケージを使う場合、この準備は不要です。

---

## 1. Ver 4.0 の導入

### A. Debianパッケージを使う（推奨・最短）

Ver 4.0 では、公式からDebianパッケージが提供されています。**Ubuntu 24.04 LTS on WSL でビルド・テストされたもの**で、ソースからのコンパイルが不要です。

導入手順は公式のダウンロードページにある「Debian package for Ubuntu-based Linux users including Windows/WSL users」の案内に従ってください。

macOSや非Ubuntu系のLinuxを使う場合は、同じくダウンロードページからDockerイメージが提供されています。Apple Silicon搭載Macでの導入手順は、OpenMX Forumに専用のスレッドがあります。

> HPCでの本番運用や、ソースコードを改変する予定がある場合は、次のソースビルドを選んでください。

### B. ソースからビルドする

作業用のディレクトリを作成します。

```
mkdir ~/DFT
cd ~/DFT
```

ダウンロードページから Ver 4.0 のソース（`openmx4.0.tar.gz`）とパッチを取得し、展開します。

```
wget （ダウンロードページで確認したURL）
tar xvfz openmx4.0.tar.gz
cd openmx4.0
ls
```

以下のディレクトリが含まれます。

| ディレクトリ | 内容 |
|---|---|
| `source` | プログラムのソースファイル |
| `work` | サンプル入力ファイル |
| （基底関数・擬ポテンシャル） | Ver 3.9 では `DFT_DATA19`。**Ver 4.0での名称は実際の展開結果を確認してください** |

パッチ（4.0.1）を適用します。適用方法は配布物に含まれる `README.txt` に従ってください。

ビルドします。

```
cd source
make -j 4        # 数字は使用するCPUコア数
make install
```

> **要確認**: makefile のコンパイラ指定については、Ver 4.0 の `README.txt` の記載を優先してください。Ver 3.9.9 では後述のとおり `mpiicx` / `mpiifx` とMKLのリンク指定が必要でしたが、Ver 4.0 での既定設定は未検証です。

---

## 2. Ver 3.9.9 の導入

既存環境の再現や、過去の計算と条件を揃える必要がある場合の手順です。

### ダウンロードと展開

```
mkdir ~/DFT
cd ~/DFT
```

ダウンロードページの「Vers. 3.9 and 3.8」から Ver 3.9 のソースとパッチのリンクを確認し、取得します。

```
wget http://t-ozaki.issp.u-tokyo.ac.jp/openmx3.9.tar.gz
wget http://www.openmx-square.org/bugfixed/21Oct17/patch3.9.9.tar.gz
```

Intel oneAPI に対応させるための追加パッチ（伊藤篤史先生ご提供）も取得します。

```
wget https://github.com/atsushi-m-ito/openmx-patch-oneapi/archive/refs/tags/v1.tar.gz
```

> **注意**: この追加パッチは **Ver 3.9.9 専用**です。Ver 4.0 には適用しないでください。適用の背景は[こちらの記事](https://qiita.com/pochman/items/1a7b80107850e027ad31)で解説されています。

展開します。

```
tar xvfz openmx3.9.tar.gz
cd openmx3.9
ls
```

- `DFT_DATA19` — 基底関数・擬ポテンシャル
- `source` — プログラムのソースファイル
- `work` — サンプル入力ファイル

### パッチの適用

パッチファイルを `patch` ディレクトリにまとめます。

```
mkdir patch
mv ../patch3.9.9.tar.gz ./patch/
mv ../v1.tar.gz ./patch/
```

Ver 3.9 用の `source` に公式パッチを当てて `source3.9.9` を作成します。

```
cp -rp source source3.9.9
cd source3.9.9
tar xvfz ../patch/patch3.9.9.tar.gz
mv kpoint.in ../work/
```

さらに oneAPI 対応パッチを当てた `source3.9.9-v1` を作成します。

```
cd ../
cp -rp source3.9.9 source3.9.9-v1
cd source3.9.9-v1
tar xvfz ../patch/v1.tar.gz --strip-components 1
```

### ビルド

```
make -j 4        # 数字は使用するCPUコア数
make install
```

エラーが出る場合は、環境（`source /opt/intel/oneapi/setvars.sh`）が読み込まれているか確認してください。

<details>
<summary>参考: パッチを使わずmakefileを手で編集する場合</summary>

上記の oneAPI 対応パッチを適用すれば、この作業は不要です（2025年7月時点）。パッチを使わない場合は、`source3.9.9/makefile` を以下のように編集します。

```
CC  = mpiicx -O3 -xHOST -fiopenmp -fcommon -Wno-error=implicit-function-declaration -I${MKLROOT}/include/fftw
FC  = mpiifx -O3 -xHOST -fiopenmp
LIB = -L${MKLROOT}/lib/intel64 -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core -lifcore -lmkl_blacs_intelmpi_lp64 -liomp5 -lpthread -lm -ldl
```

`${MKLROOT}` は `setvars.sh` によって設定される環境変数です。バージョン番号を含むパスを直接書くと、oneAPIを更新するたびに修正が必要になります。

</details>

---

## 3. 動作テスト（共通）

`work` ディレクトリに移動し、付属のテストセットを実行します。

```
cd ~/DFT/openmx4.0/work/      # Ver 3.9.9 の場合は openmx3.9/work/
mpirun -np 8 ./openmx -runtest -nt 2 < /dev/null > log.txt 2>&1 &
```

計算結果と参照値の差分が `runtest.result` に出力されます。差が十分小さければ成功です。

### 並列数の指定について

`-np` はMPIプロセス数、`-nt` は各プロセスが使うOpenMPスレッド数です。**両者の積が物理コア数を超えないようにしてください。** 超えると性能が大きく低下します。

```
# 16コアの環境での例
mpirun -np 8 ./openmx -runtest -nt 2 ...   # 8 × 2 = 16
mpirun -np 16 ./openmx -runtest -nt 1 ...  # 16 × 1 = 16
```

コア数は `nproc` で確認できます。

---

## 4. 計算の実行（共通）

コンパイルで生成された実行ファイル `openmx` と、入力ファイル（`xx.dat`）を同じディレクトリに置いて実行します。

```
mpirun -np 8 ./openmx xx.dat -nt 2 < /dev/null > log.txt 2>&1 &
```

末尾の `&` によりバックグラウンドで実行されます。

実行中のログは以下で確認します。終了するには `Ctrl+C` を押します（計算は継続します）。

```
tail -F log.txt
```

ジョブの状態は以下で確認・操作できます。

```
jobs        # 状況確認
bg %1       # 停止中のジョブ番号1を再開する
kill %1     # 実行中のジョブ番号1を停止する
```

SSH接続を切断しても計算を継続させたい場合は、`tmux` の利用をおすすめします。詳細は[計算サーバへの接続方法](../../Server_setting/Howtoaccess_server/README.md)を参照してください。

---

## 次に読むページ

- [DFT計算1（インプットファイルの作成）](../OpenMX_calculation/README.md)
- [DFT計算2（計算の実行）](../OpenMX_calculation2/README.md)
- [阪大スパコンSQUIDでのコンパイル](../OpenMX_installation2/README.md)
