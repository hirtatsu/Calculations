# LAMMPSのインストール（基本版）

最終更新: 2026-08 ／ 対象: LAMMPS 安定版、Ubuntu（WSL2 / ネイティブ）、Intel oneAPI

分子動力学計算ソフトウェア LAMMPS を、CPU並列で動作する構成でビルドします。**初めてLAMMPSをビルドする場合はこのページから始めてください。**

GPU（CUDA）、Kokkos、AMD APU を使う構成や、複数のバイナリを作り分ける方法は[LAMMPS環境構築ガイド（統合版）](../LAMMPS_installation2/README.md)を参照してください。

---

## 0. 前提: コンパイラとMPI環境

Intel oneAPI（コンパイラ、Intel MPI、MKL）が必要です。導入手順は[MPI環境の構築（Intel oneAPI）](../../Setting_WSL/MPI/README.md)を参照してください。

研究室の計算サーバには導入済みのため、環境を読み込むだけで構いません。

```
source /opt/intel/oneapi/setvars.sh
icx -v
```

シェルの設定で自動的に読み込むようにしている場合は、この操作は不要です。

ビルドに必要なパッケージも入れておきます。

```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential cmake -y
```

---

## 1. ソースの取得

作業用のディレクトリを作成します。

```
cd
mkdir MD
cd MD
```

安定版のソースを取得します。`lammps-stable.tar.gz` は常に最新の安定版を指すため、URLにバージョンを含める必要はありません。

```
wget https://download.lammps.org/tars/lammps-stable.tar.gz
```

展開する前に、中身のディレクトリ名（＝バージョン）を確認します。

```
tar tzf lammps-stable.tar.gz | head -1
```

`lammps-22Jul2025/` のように表示されます。展開して、そのディレクトリに移動します。

```
tar xvzf lammps-stable.tar.gz
cd lammps-22Jul2025        # 上で確認したディレクトリ名に読み替えてください
```

> 2026年8月時点の安定版は 22 Jul 2025（update 5）です。以降の手順では、このディレクトリを起点とします。

---

## 2. ビルド

`build` ディレクトリを作成して移動します。

```
mkdir build
cd build
```

cmakeで設定を行います。ここでは MANYBODY、VORONOI、MEAM、REAXFF の各パッケージを有効にしています。

```
cmake -C ../cmake/presets/most.cmake \
-D CMAKE_C_COMPILER=icx \
-D CMAKE_CXX_COMPILER=icpx \
-D CMAKE_Fortran_COMPILER=ifx \
-D FFT=MKL \
-D BUILD_MPI=yes \
-D PKG_MANYBODY=yes \
-D PKG_VORONOI=yes \
-D DOWNLOAD_VORO=yes \
-D PKG_REAXFF=yes \
-D PKG_MEAM=yes ../cmake
```

各指定の意味は以下のとおりです。

| 指定 | 内容 |
|---|---|
| `-C ../cmake/presets/most.cmake` | よく使われるパッケージ群をまとめて有効にするプリセット |
| `CMAKE_*_COMPILER` | oneAPIのコンパイラ（`icx` / `icpx` / `ifx`）を使用 |
| `FFT=MKL` | 高速フーリエ変換にIntel MKLを使用 |
| `BUILD_MPI=yes` | MPI並列を有効化 |
| `PKG_*` | 使用するポテンシャルや機能のパッケージを個別に有効化 |
| `DOWNLOAD_VORO` | VORONOIパッケージが必要とするライブラリを自動取得 |

必要なパッケージは計算内容によって異なります。一覧は[LAMMPS公式ドキュメント](https://docs.lammps.org/Build_package.html)を参照してください。

ビルドしてインストールします。

```
make -j 4        # 数字は使用するCPUコア数
make install
```

実行ファイル `lmp` が `~/.local/bin/` に配置されます。

> 複数の構成（CPU版・GPU版など）を使い分けたい場合は、cmakeに `-D LAMMPS_MACHINE=cpu` のように指定すると `lmp_cpu` という名前で生成されます。詳しくは[統合版](../LAMMPS_installation2/README.md)を参照してください。

---

## 3. PATHを通す

```
cd
vim .bashrc
```

最終行に以下を追加します。

```
export PATH=~/.local/bin:$PATH
```

反映させます。

```
source ~/.bashrc
```

確認します。バージョン情報などが表示されれば成功です。

```
lmp -help | head
```

---

## 4. お試し計算（melt）

LAMMPSに付属している計算例を実行してみます。

### 入力ファイルの編集

```
cd ~/MD/lammps-22Jul2025/examples/melt
vim in.melt
```

原子の座標を出力させるため、`dump` で始まる行のコメント記号（`#`）を外します。

```
dump            id all atom 50 dump.melt
```

> 行番号はバージョンによって変わります。`/dump` で検索して該当行を探してください（vimでは `/dump` と入力してEnter）。

### 実行

シングルコアで実行する場合:

```
lmp -in in.melt
```

マルチコア（例: 4コア）で実行する場合:

```
mpirun -np 4 lmp -in in.melt
```

画面に計算の進捗が流れ、最後に `Total wall time: xx:xx:xx` と表示されれば成功です。

長時間の計算をSSH接続経由で実行する場合は、`tmux` を使うと接続を切断しても計算が継続します。詳細は[計算サーバへの接続方法](../../Server_setting/Howtoaccess_server/README.md)を参照してください。

```
tmux new -s calc

# セッション内で実行（画面に流しながらログも保存）
mpirun -np 4 lmp -in in.melt 2>&1 | tee log.melt

# Ctrl+b を押してから d でセッションを抜ける
```

### 結果の確認

```
ls
```

以下のファイルが生成されているはずです。

| ファイル | 内容 |
|---|---|
| `dump.melt` | 各時間ステップにおける原子の配置（可視化に使用） |
| `log.lammps` | 計算のログ |

---

## 5. 可視化

`dump.melt` は OVITO で開くことができます。

- [OVITO](https://www.ovito.org/) — Basic版は無償で利用できます（Pro版は有償）

使い方は[OVITOの使い方メモ](../OVITO_tips/README.md)を参照してください。

---

## 次に読むページ

- [LAMMPS環境構築ガイド（統合版）](../LAMMPS_installation2/README.md) — GPU・Kokkos・AMD APUを使う場合
- [Atomskのインストールと使い方](../Atomsk_installation/README.md) — 計算モデルの作成
- [OVITOの使い方メモ](../OVITO_tips/README.md) — 計算結果の可視化
