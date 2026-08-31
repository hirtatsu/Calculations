# LAMMPS + MACE 実行環境の構築（ML-IAPインターフェース）

最終更新: 2026-08 ／ 対象: LAMMPS develop（commit 6404ba2）、mace-torch 0.3.16、CUDA 12.6、Ubuntu 22.04

機械学習ポテンシャル（MLIP）の一種であるMACEをLAMMPSから実行するための環境を構築します。EAM・MEAM・ReaxFFなどの古典ポテンシャルとACE（ML-PACE）も同じバイナリに同梱するため、これ1本が実質的なメインビルドになります。

> **前提となるページ**: [GPGPU環境の構築](../../Setting_WSL/GPGPU/README.md)（ドライバとCUDA Toolkit）、
> [Pythonの環境構築](../../Setting_WSL/Python_installation2/README.md)。
> 環境変数は `.bashrc` に書かず、用途別の環境スクリプトで明示的に読み込む方針を前提とします
> （[LAMMPS環境構築ガイド（統合版）](../LAMMPS_installation2/README.md)の0節を参照）。

---

## ルートの選定

MACEをLAMMPSに接続する方法は2つあります。

1. **pair_style mace 方式**: MACE側が管理するLAMMPSフォークをlibtorchとリンクして組む。フォークが本家に追随しきれず陳腐化しやすい。
2. **ML-IAP方式**（本ページ）: 本家LAMMPSのdevelopブランチをPython結合付きで組み、mliap形式に変換したモデルを読む。GPU実行に最適化されており、cuEquivarianceによるカーネル高速化もこちらのみ。

MACE公式ドキュメントもML-IAP方式の性能優位を明言しているため、本ページでは2を採用します。ML-IAPインターフェースはstable版では不足があり、**developブランチが必要**です。

---

## 検証環境

| 項目 | 内容 |
|---|---|
| ハードウェア | Intel Xeon Gold 6312U（24C, Ice Lake-SP）+ NVIDIA RTX 6000 Ada（48GB） |
| OS | Ubuntu 22.04.5 LTS |
| NVIDIAドライバ | 580.173.02 |
| CUDA Toolkit | 12.6（`/usr/local/cuda-12.6`、環境スクリプトで明示） |
| コンパイラ | GCC 11.4.0 + OpenMPI（Ubuntu標準） |
| LAMMPS | develop、commit `6404ba2`（version 4 Jul 2026 - Development） |
| Python | 3.10.12（venv） |
| torch / mace-torch | 2.13.0+cu130 / 0.3.16 |
| 確認日 | 2026/08/31 |

> **CUDAバージョンの混成について**: 本構成では、LAMMPS本体はシステムのCUDA 12.6で、
> torchはwheel同梱のCUDA 13.0ランタイムで動きます。**意図的な混成**です。
> 双方ともドライバ（580系）にのみ依存するため、同一プロセス内で共存できることを
> 実走で確認済みです。MACE公式ドキュメントは `torch<=2.5.0` を推奨していますが、
> これはモデル読み込み時の警告回避が目的であり、2.13系でも動作します
> （deprecation系の警告が多数出ますが無害です）。

---

## 1. Python環境（venv）の構築

素のシェル（setvarsもCUDAも読み込んでいない状態）から始めます。

```bash
sudo apt install -y openmpi-bin libopenmpi-dev libfftw3-dev libblas-dev liblapack-dev python3-venv

mkdir -p ~/envs
python3 -m venv ~/envs/mace
source ~/envs/mace/bin/activate
pip install -U pip wheel

pip install torch
pip install mace-torch
pip install cuequivariance cuequivariance-torch cuequivariance-ops-torch-cu12 cupy-cuda12x
pip install cython cmake
```

- **cython は必須**です。ML-IAPのPython結合のグルーコード生成に使われ、無いとcmakeが
  `Could NOT find Cythonize` で止まります。
- **cmake** はUbuntu 22.04標準の3.22では新しいKokkosの要求を満たさないため、venv内に
  新しい版（4系）を入れます。
- `cuequivariance-ops-torch-cu12` はtorch本体がcu13系でもimportできることを確認済み。
  実行時にカーネルエラーが出る場合のみ `-cu13` 版へ差し替えてください。

動作確認:

```bash
python -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0))"
pip freeze > ~/envs/mace-freeze-$(date +%Y%m%d).txt   # 環境の記録
```

---

## 2. LAMMPS（develop）の取得

```bash
cd ~/MD
git clone --depth=1 https://github.com/lammps/lammps.git lammps-develop
cd lammps-develop
git rev-parse --short HEAD   # ← このcommitハッシュを必ず控える（再現性の要）
```

developブランチは日々変わるため、**いつの状態で組んだかはcommitハッシュでしか特定できません**。本ページの検証は `6404ba2` で行いました。

---

## 3. CMake構成

環境スクリプトとvenvをこの順で読み込みます（oneAPIのsetvarsは**読み込まない**。GNU系で組みます）。

```bash
source ~/env/cuda126.sh
source ~/envs/mace/bin/activate
mkdir build-mace && cd build-mace
```

```bash
cmake \
 -D CMAKE_BUILD_TYPE=Release \
 -D LAMMPS_MACHINE=mace \
 -D CMAKE_INSTALL_PREFIX=$HOME/.local \
 -D BUILD_MPI=yes \
 -D BUILD_OMP=yes \
 -D BUILD_SHARED_LIBS=yes \
 -D PKG_KOKKOS=yes \
 -D Kokkos_ENABLE_CUDA=yes \
 -D Kokkos_ENABLE_OPENMP=yes \
 -D Kokkos_ARCH_ICX=yes \
 -D Kokkos_ARCH_ADA89=yes \
 -D FFT_KOKKOS=CUFFT \
 -D PKG_PYTHON=yes \
 -D PKG_ML-IAP=yes \
 -D PKG_ML-SNAP=yes \
 -D MLIAP_ENABLE_PYTHON=yes \
 -D Python_EXECUTABLE=$HOME/envs/mace/bin/python \
 -D PKG_MANYBODY=yes \
 -D PKG_MEAM=yes \
 -D PKG_REAXFF=yes \
 -D PKG_QEQ=yes \
 -D PKG_MOLECULE=yes \
 -D PKG_KSPACE=yes \
 -D PKG_RIGID=yes \
 -D PKG_ML-PACE=yes -D DOWNLOAD_PACE=yes \
 -D PKG_VORONOI=yes -D DOWNLOAD_VORO=yes \
 -D PKG_EXTRA-COMPUTE=yes -D PKG_EXTRA-FIX=yes -D PKG_EXTRA-DUMP=yes \
 ../cmake
```

要点:

- `LAMMPS_MACHINE=mace` でバイナリ名を `lmp_mace` にし、既存ビルドと共存させる。
- `BUILD_SHARED_LIBS=yes` はPythonモジュール（`make install-python`）の前提。
- `Kokkos_ARCH_*` は**マシンの実構成に合わせて必ず変更**（本例はIce Lake + Ada世代）。
- `MLIAP_ENABLE_PYTHON=yes` と `Python_EXECUTABLE`（venvのpythonを指定）が本ページの核。
- `FFT_KOKKOS=CUFFT` を指定しないとKISS FFTになり、GPU上のKSPACE計算が遅くなる。

出力サマリのDefinesに **`MLIAP_PYTHON`** が含まれていること、`Kokkos is configured ...
(using NVIDIA version 12.6.85)` のようにCUDAが意図した版であることを確認してから
次へ進みます。

---

## 4. ビルドとインストール

30〜60分かかるためtmux内での実行を推奨します。

```bash
cmake --build . -j 20 2>&1 | tee build.log
make install
make install-python   # lammpsのPythonパッケージ（wheel）をvenvへインストール
```

### 環境スクリプトの作成

共有ライブラリ（`~/.local/lib/liblammps_mace.so.0`）に実行時パスを通す必要があります。
これを含めた環境スクリプトを1本作り、以後 `lmp_mace` はこれを読んでから使います。

```bash
cat > ~/env/lammps-mace.sh << 'EOF'
# lmp_mace (develop/6404ba2, gcc+Kokkos/CUDA12.6+ML-IAP Python, 2026-08-31)
# MACE・古典・ACE用メインビルド。mace venvと一体。非対話シェルでも自己完結
source ~/env/cuda126.sh
source ~/envs/mace/bin/activate
export PATH=$HOME/.local/bin:$PATH
export LD_LIBRARY_PATH=$HOME/.local/lib:$LD_LIBRARY_PATH
EOF
```

### 起動確認

```bash
source ~/env/lammps-mace.sh
lmp_mace -h | head -3
python -c "import lammps; print(lammps.__file__)"   # venv配下のパスが返ればOK
```

---

## 5. MACEモデルの入手とmliap形式への変換

基盤モデルMACE-MP（small）で疎通確認します。

```bash
mkdir -p ~/MLIP/models && cd ~/MLIP/models

# ダウンロード（~/.cache/mace/ にキャッシュされる）
python -c "from mace.calculators import mace_mp; mace_mp(model='small')"
ls ~/.cache/mace/

# キャッシュから取り出してML-IAP形式へ変換（ファイル名は上のlsに合わせる）
cp ~/.cache/mace/20231210mace128L0_energy_epoch249model ./mace-mp0-small.model
mace_create_lammps_model mace-mp0-small.model --format=mliap
# → mace-mp0-small.model-mliap_lammps.pt が生成される
```

自前で学習したモデルも同じコマンドで変換できます。

---

## 6. 実行テスト（Cu fcc, 256原子, NVT 100ステップ）

```bash
mkdir -p ~/MLIP/test && cd ~/MLIP/test
cat > in.mace-test << 'EOF'
units           metal
atom_style      atomic
atom_modify     map yes
newton          on

lattice         fcc 3.615
region          box block 0 4 0 4 0 4
create_box      1 box
create_atoms    1 box
mass            1 63.55

pair_style      mliap unified ../models/mace-mp0-small.model-mliap_lammps.pt 0
pair_coeff      * * Cu

velocity        all create 300 4928459
fix             1 all nvt temp 300 300 0.1
thermo          10
run             100
EOF

lmp_mace -k on g 1 -sf kk -pk kokkos newton on neigh half -in in.mace-test
```

- `pair_style mliap unified <モデル> 0` の末尾の `0` は必須の引数。
- `pair_coeff * * Cu` の元素リストは、モデルの学習元素の部分集合を
  LAMMPSの型順に並べる。
- 実行フラグ `-k on g 1 -sf kk` でGPU 1基のKokkos実行になる。

検証時の結果（初回実行、JITウォームアップ込み）: 100ステップのLoop timeが7.05秒、
3.63 katom-step/s。ネイバーリストは `pair mliap/kk (kokkos_device)` としてGPU側で
構築される。256原子はGPUには小さすぎる系で、Python呼び出しのオーバーヘッドが
支配的なため、**この数値から性能を判断しないこと**。実性能は数千原子以上の系で
測る。

---

## つまずきと対処（実際に遭遇したもの）

| 症状 | 原因と対処 |
|---|---|
| cmakeが `Could NOT find Cythonize` で停止 | venvに `pip install cython` |
| `lmp_mace: error while loading shared libraries: liblammps_mace.so.0` | 共有ライブラリ版のため `LD_LIBRARY_PATH` に `~/.local/lib` が必要。環境スクリプト（上記）を読む |
| cmakeで `cmake_minimum_required` 系のエラー | cmake 4系は古い外部ライブラリのビルド設定と非互換。`-D CMAKE_POLICY_VERSION_MINIMUM=3.5` を追加 |

---

## 運用メモ

- 本ビルドはMACE（ML-IAP）と古典・ACEを兼ねるメインビルドです。既存のIntel+MKLビルド
  （`lmp_kokkos` 等）を残す場合は、環境スクリプトを対にして使い分けます
  （例: `~/env/lammps-legacy.sh` はsetvarsを読む、`~/env/lammps-mace.sh` は読まない）。
- 複数MPIランクでKokkos実行する場合、UbuntuのOpenMPIはGPU-awareではないため
  `-pk kokkos gpu/aware off` を付けてください（GPU 1基・1ランクなら不要）。
- developブランチのビルドを更新する場合は、必ず新しいcommitハッシュを控え、
  旧バイナリを凍結してから入れ替えること。

## 関連ページ

- [LAMMPS環境構築ガイド（統合版）](../LAMMPS_installation2/README.md) — 古典ポテンシャル中心のビルド
- [GPGPU環境の構築](../../Setting_WSL/GPGPU/README.md)
- [Pythonの環境構築（Miniforge）](../../Setting_WSL/Python_installation2/README.md)
- [MACE公式ドキュメント（LAMMPS ML-IAP）](https://mace-docs.readthedocs.io/en/latest/guide/lammps_mliap.html)
