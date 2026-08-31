# LAMMPS環境構築ガイド（CPUサーバー / NVIDIA GPU / Kokkos / AMD APU 統合版）

研究室で運用する複数の計算環境に対して、LAMMPSをソースからビルドする手順をまとめる（2026/8）。

> **初めてLAMMPSをビルドする場合**は、CPU版1構成に絞った[LAMMPSのインストール（基本版）](../LAMMPS_installation/README.md)から始めるとよい。
> 本ページは、GPU・Kokkos・AMD APUを含む複数構成を作り分けるための統合ガイドである。

## 対象環境と作り分け

| 記号 | 環境 | 代表ハード | バイナリ名 |
|---|---|---|---|
| A | CPU専用サーバー（Intel oneAPI） | Xeon 6952P（96コア、AVX-512） | `lmp_cpu` |
| B | NVIDIA GPU（GPUパッケージ、CUDA） | RTX 3060 / A4000 / 4090 / 6000 Ada | `lmp_gpu` |
| C | NVIDIA GPU（Kokkos + CUDA） | w7-2495X + RTX 6000 Ada | `lmp_kokkos` |
| D | AMD APU（GPUパッケージ、HIP） | Instinct MI300A（プラズマシミュレータ） | `lmp_gpu` |
| E | AMD APU（Kokkos + HIP） | Instinct MI300A（プラズマシミュレータ） | `lmp_kokkos` |

`LAMMPS_MACHINE` でバイナリ名を分けているので、同一マシンに複数構成を共存できる
（buildディレクトリも構成ごとに分ける: `build-cpu`, `build-gpu`, `build-kokkos` など）。

---

## 0. 事前準備

### 共通パッケージ

```bash
sudo apt install -y cmake build-essential ccache gfortran python3-dev python3-pip \
                    libjpeg-dev libpng-dev libhdf5-serial-dev hdf5-tools ffmpeg
```

- GNUツールチェーンでビルドする場合はさらに
  `openmpi-bin libopenmpi-dev libfftw3-dev libblas-dev liblapack-dev` を追加する。
- Intel oneAPI（環境A・B・C）を使う場合、コンパイラ・MPI・MKL（FFT/BLAS/LAPACK）は
  oneAPI側を使うため上記GNU系ライブラリは不要。シェルで環境を読み込んでおく:

```bash
source /opt/intel/oneapi/setvars.sh
which icx icpx ifx mpiicx   # すべてパスが返ることを確認
```

> **⚠️ oneAPIを使ったビルドの依存関係（ビルド時・実行時とも）**
>
> setvars済み環境でビルドしたバイナリは、MKLやIntel MPIの共有ライブラリに**実行時にも**依存する。
> setvarsを読んでいないシェル（ジョブスクリプト、cron、非対話ssh）から起動すると
>
> ```
> error while loading shared libraries: libmkl_intel_lp64.so.2:
> cannot open shared object file
> ```
>
> のようなエラーで落ちる。対話シェルでは動くのにバッチ経由では落ちる場合、まずこれを疑う。
> ジョブスクリプトの冒頭にも `source /opt/intel/oneapi/setvars.sh > /dev/null` を書くこと。
>
> なお、setvarsやCUDAの環境変数を `.bashrc` に書いて常時読み込む運用は推奨しない。
> ビルドが意図しないコンパイラ・ライブラリを拾う原因になる（本ガイドの構成A〜Cを
> 同一マシンで作り分ける場合は特に）。用途別の環境スクリプト（例: `~/env/cuda126.sh`、
> `~/env/oneapi.sh`）を用意し、ビルド・実行の際に明示的に `source` する方式をとる。

### NVIDIA GPU環境（B・C）

- WSL2の場合: CUDA on WSLのセットアップは [こちら](../../Setting_WSL/GPGPU/README.md) と
  [公式マニュアル](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#abstract)
- ネイティブLinuxの場合: NVIDIAドライバとCUDA Toolkitを導入し `nvidia-smi` と `nvcc --version` を確認
- 複数バージョンのCUDA Toolkitを併置している場合は、symlink（`/usr/local/cuda`）に依存せず、
  環境スクリプトで `CUDA_HOME` とPATH/LD_LIBRARY_PATHを版込みで明示する:

```bash
# 例: ~/env/cuda126.sh
export CUDA_HOME=/usr/local/cuda-12.6
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

### AMD APU環境（D・E）

プラズマシミュレータではmoduleで環境を読み込む:

```bash
module load openmpi/5.0.7/rocm6.3.3
# Eの場合はさらに
module load openmpi/5.0.7/rocm6.3.3_amdflang_afar
export OMPI_CC=amdclang
```

---

## 1. ソースの取得（共通）

```bash
mkdir -p ~/MD && cd ~/MD
wget https://download.lammps.org/tars/lammps-stable.tar.gz
tar tzf lammps-stable.tar.gz | head -1   # 展開前にバージョン（ディレクトリ名）を確認
tar xvzf lammps-stable.tar.gz
cd lammps-*   # 展開されるディレクトリ名はバージョンで変わる（例: lammps-22Jul2025）
```

- `lammps-stable.tar.gz` は常に最新安定版を指す。**2026年8月時点の安定版は 22 Jul 2025（update 5）**。
  安定版にはバグ修正が同一版のupdateとして随時取り込まれる。
- 最新機能が必要な場合のみ feature release（`https://download.lammps.org/tars/lammps.tar.gz`）を使う。
  feature版への修正は次のfeature版でしか供給されない点に注意。

```bash
mkdir build-<構成名> && cd build-<構成名>
```

---

## 2. CMake構成（環境別）

CMakeプリセットの一覧は[公式ドキュメント](https://docs.lammps.org/Build_package.html#cmake-presets-for-installing-many-packages)を参照。
以下では研究で使うパッケージ（MANYBODY, MEAM, REAXFF, VORONOI ほか）を明示的に指定する。

### A. CPU専用（Intel oneAPI、AVX-512）

検証環境: TSVR-Juventus（Xeon 6952P 96コア / Granite Rapids、Ubuntu 24.04.4、
oneAPI 2025.3.2、LAMMPS 22Jul2025 update 5、2026/8/29）

```bash
cmake \
 -D CMAKE_C_COMPILER=icx \
 -D CMAKE_CXX_COMPILER=icpx \
 -D CMAKE_Fortran_COMPILER=ifx \
 -D MPI_C_COMPILER=mpiicx \
 -D MPI_CXX_COMPILER=mpiicpx \
 -D CMAKE_BUILD_TYPE=Release \
 -D CMAKE_TUNE_FLAGS="-xHOST -qopt-zmm-usage=high" \
 -D LAMMPS_MACHINE=cpu \
 -D CMAKE_INSTALL_PREFIX=$HOME/.local \
 -D BUILD_MPI=yes \
 -D BUILD_OMP=yes \
 -D FFT=MKL \
 -D BLA_VENDOR=Intel10_64lp \
 -D PKG_MANYBODY=yes \
 -D PKG_MEAM=yes \
 -D PKG_REAXFF=yes \
 -D PKG_QEQ=yes \
 -D PKG_MOLECULE=yes \
 -D PKG_KSPACE=yes \
 -D PKG_RIGID=yes \
 -D PKG_REPLICA=yes \
 -D PKG_MC=yes \
 -D PKG_EXTRA-COMPUTE=yes \
 -D PKG_EXTRA-FIX=yes \
 -D PKG_EXTRA-DUMP=yes \
 -D PKG_VORONOI=yes \
 -D DOWNLOAD_VORO=yes \
 -D PKG_ML-SNAP=yes \
 -D PKG_ML-IAP=yes \
 -D PKG_ML-PACE=yes \
 -D DOWNLOAD_PACE=yes \
 -D PKG_OPENMP=yes \
 -D PKG_OPT=yes \
 ../cmake
```

- `-xHOST` で実行マシンのISA（AVX-512）に最適化、`-qopt-zmm-usage=high` でZMMレジスタを積極利用。
- FFT/BLAS/LAPACKはすべてMKL。KSPACE系はスレッド版MKL FFTで動く。
- ML-SNAP / ML-IAP / ML-PACE はMLIP（SNAP・ACE）の受け皿として同梱。
- **PKG_INTELは有効化しない。** `pair_tersoff_intel.cpp` がicx 2025.3系＋`-xHOST` で
  ビルド不能（SSE intrinsicsのターゲット属性検査がLLVM系で厳格化されたことに起因）。
  Tersoff系は素の版（MANYBODY）とomp版で提供されるため機能上の欠落はない。
  INTELパッケージのEAM高速化を検証したい場合はfeature版（4 Jul 2026以降、
  Xeon Phi対応削除後）で別ビルドを試すこと。

### B. NVIDIA GPU（GPUパッケージ、CUDA）

`GPU_ARCH` は[NVIDIA公式のCompute Capability一覧](https://developer.nvidia.com/cuda-gpus)で確認。
RTX 3060 / A4000 → `sm_86`、RTX 4090 / 6000 Ada → `sm_89`。

```bash
cmake -C ../cmake/presets/most.cmake \
 -D LAMMPS_MACHINE=gpu \
 -D CMAKE_C_COMPILER=icx \
 -D CMAKE_CXX_COMPILER=icpx \
 -D CMAKE_Fortran_COMPILER=ifx \
 -D FFT=MKL \
 -D PKG_GPU=yes \
 -D GPU_API=cuda \
 -D GPU_ARCH=sm_89 \
 -D BUILD_MPI=yes \
 -D PKG_MANYBODY=yes \
 -D PKG_VORONOI=yes \
 -D DOWNLOAD_VORO=yes \
 -D PKG_MEAM=yes \
 -D PKG_REAXFF=yes \
 ../cmake
```

- nvccとIntelコンパイラの組み合わせで失敗する場合は、コンパイラ指定3行と `FFT=MKL` を外し
  GNUツールチェーンでビルドする（要 `libfftw3-dev` 等）。

### C. NVIDIA GPU（Kokkos + CUDA）

検証環境: w7-2495X + RTX 6000 Ada、Ubuntu 24.04、oneAPI 2025.1.0、CUDA Toolkit 13.1。
ホスト/GPUのアーキテクチャ指定は[公式ドキュメント](https://docs.lammps.org/Build_extras.html#kokkos)で確認
（例: Sapphire Rapids → `SPR`、Ada世代 CC8.9 → `ADA89`）。

```bash
cmake -C ../cmake/presets/most.cmake \
 -D LAMMPS_MACHINE=kokkos \
 -D PKG_KOKKOS=yes \
 -D Kokkos_ARCH_SPR=yes \
 -D Kokkos_ARCH_ADA89=yes \
 -D Kokkos_ENABLE_CUDA=yes \
 -D Kokkos_ENABLE_OPENMP=yes \
 -D BUILD_OMP=yes \
 -D CMAKE_CXX_COMPILER=`which mpicxx` \
 -D MPI_C_COMPILER=`which mpicc` \
 -D PKG_MEAM=yes \
 -D PKG_MANYBODY=yes \
 -D PKG_VORONOI=yes \
 -D DOWNLOAD_VORO=yes \
 -D PKG_REAXFF=yes \
 ../cmake
```

- `Kokkos_ARCH_*` は**ビルドするマシンの実構成に合わせて必ず変更する**
  （上記はSapphire Rapids + Ada世代の例）。
- `which mpicxx` は読み込み済みの環境に依存する。**oneAPI（setvars）環境ではIntel MPIの
  ラッパーを指し、生成物は0節の実行時依存（MKL・Intel MPI）を持つ**。GNUツールチェーンで
  組みたい場合はOpenMPIを導入し、setvarsを読んでいないシェルでcmakeを実行する。

### D. AMD APU（GPUパッケージ、HIP）

```bash
cmake -C ../cmake/presets/most.cmake \
 -D LAMMPS_MACHINE=gpu \
 -D PKG_GPU=yes \
 -D GPU_API=HIP \
 -D HIP_ARCH=gfx942 \
 -D CMAKE_CXX_COMPILER=hipcc \
 -D CMAKE_CXX_FLAGS="-mcmodel=large --offload-arch=gfx942" \
 -D BUILD_MPI=yes \
 -D PKG_MANYBODY=yes \
 -D PKG_VORONOI=yes \
 -D DOWNLOAD_VORO=yes \
 -D PKG_MEAM=yes \
 -D PKG_REAXFF=yes \
 ../cmake
```

### E. AMD APU（Kokkos + HIP）

```bash
cmake -C ../cmake/presets/most.cmake \
 -D LAMMPS_MACHINE=kokkos \
 -D PKG_KOKKOS=yes \
 -D Kokkos_ARCH_ZEN4=yes \
 -D Kokkos_ARCH_AMD_GFX942_APU=yes \
 -D Kokkos_ENABLE_HIP=yes \
 -D Kokkos_ENABLE_OPENMP=yes \
 -D BUILD_OMP=yes \
 -D BUILD_MPI=yes \
 -D CMAKE_CXX_COMPILER=hipcc \
 -D CMAKE_CXX_FLAGS="-fopenmp -mcmodel=large --offload-arch=gfx942" \
 -D PKG_MEAM=yes \
 -D PKG_MANYBODY=yes \
 -D PKG_VORONOI=yes \
 -D DOWNLOAD_VORO=yes \
 -D PKG_REAXFF=yes \
 ../cmake
```

---

## 3. ビルドとインストール（共通）

```bash
cmake --build . -j 32     # 並列数はマシンのコア数に応じて（時間がかかるのでtmux推奨）
make install              # CMAKE_INSTALL_PREFIX（既定 ~/.local）の bin/ に配置される
```

### PATHの確認

Ubuntuの標準 `.bashrc` は `~/.local/bin` が存在すれば自動でPATHに追加する。
`which lmp_cpu` 等で見つからない場合のみ、`.bashrc` 末尾に追記して反映:

```bash
export PATH=$HOME/.local/bin:$PATH
```

（`.bashrc` に足してよいのはこのPATH追加まで。setvarsやCUDAの環境変数は
0節のとおり環境スクリプト側に置く。）

### 起動確認

```bash
lmp_cpu -h | head -5                          # バージョン表示
lmp_cpu -h | grep -A5 'Installed packages'    # パッケージ一覧
```

（構成に応じて `lmp_gpu` / `lmp_kokkos` に読み替え。oneAPIビルドの場合は
setvarsを読んだシェルで実行すること）

---

## 4. お試し計算（melt）

同梱の計算例を使う。

```bash
cd ~/MD/lammps-*/examples/melt
```

原子配置を可視化したい場合のみ、`in.melt` のdump行のコメントアウト（行頭の `#`）を外す:

```
dump            id all atom 50 dump.melt
```

**ベンチマークを取る場合はdumpを無効のままにする**（I/Oが計測を汚す）。

### 実行コマンド（環境別）

```bash
# A: CPU（純MPI、コア数まで並列化）
mpirun -n 96 lmp_cpu -in in.melt

# A: CPU（MPI×OpenMPハイブリッドの例）
mpirun -n 32 lmp_cpu -sf omp -pk omp 3 -in in.melt

# B/D: GPU単独
lmp_gpu -sf gpu -pk gpu 1 -in in.melt

# B/D: GPU＋CPUハイブリッド
mpirun -n 8 lmp_gpu -sf gpu -pk gpu 1 -in in.melt

# C/E: Kokkos（GPU 1基＋OpenMP 12スレッド）
lmp_kokkos -k on g 1 t 12 -sf kk -in in.melt
lmp_kokkos -k on g 1 t 12 -sf kk -pk kokkos newton on neigh half -in in.melt  # 追加指定で速くなる場合あり
```

最後に `Total wall time: xx:xx:xx` が表示されれば成功。
出力ファイルは `dump.melt`（原子配置、OVITOで可視化）と `log.lammps`（ログ）。

---

## 5. 計算時間の比較（meltベンチマーク）

計算内容: `examples/melt/in.melt` をもとに `region box block 0 50 0 50 0 50`、`run 2500` に変更
（50万原子・2500ステップ、dump無効）。各環境の百分率はその環境の「並列なし」を基準とする。

計算環境1: CPUにIntel Core i5 12400F、GPUにNVIDIA GeForce RTX 3060。

- 並列なしの場合: 0:06:59 (Ref.)
- 8並列の場合: 0:01:39 (-76%)
- 12並列の場合: 0:01:18 (-81%)
- GPUの場合: 0:00:29 (-93%)

計算環境2: CPUにIntel Core i7 13700KF、GPUにNVIDIA RTX A4000。

- 並列なしの場合: 0:05:40 (Ref.)
- 12並列の場合: 0:00:52 (-84%)
- 20並列の場合: 0:00:42 (-88%)
- GPUの場合: 0:00:24 (-93%)

計算環境3: CPUにIntel Core i7 13700KF、GPUにNVIDIA RTX 4090。

- GPUの場合: 0:00:19 (-94%)

計算環境4: CPUにIntel Xeon w7-2495X、GPUにNVIDIA RTX 6000 Ada。

- 並列なしの場合: 0:07:04 (Ref.)
- 24並列の場合: 0:00:28 (-93%)
- 36並列の場合: 0:00:18 (-96%)
- 42並列の場合: 0:00:16 (-96%)
- 48並列の場合: 0:00:20 (-95%)
- GPUの場合: 0:00:32 (-92%)
- CPU8並列＋GPUの場合: 0:00:09 (-94%)

計算環境5: GPUにAMD Instinct MI300A @プラズマシミュレータ。

- GPUの場合: 0:00:24 (-93%)

計算環境6: CPUにIntel Xeon 6952P（96コア、MRDIMM DDR5-8800 6ch実装）@TSVR-Juventus。

- 96並列の場合: 0:00:06（並列なし基準は未測定。全環境を通じて最速）
  - ループ時間6.40秒、195.4 Matom-step/s、CPU使用率99.5%
  - 内訳: Pair 66.6% / Neigh 13.8% / Comm 11.3% / Output 6.8%

### 考察メモ

- meltはLJポテンシャルの単純な系で、**演算律速・キャッシュに乗りやすいベンチマーク**。
  コア数とAVX-512ベクトル化が素直に効く一方、メモリ帯域やGPU転送の性格は測れない。
- 96コアCPU単独（環境6）が、CPU＋GPUハイブリッドの従来最速（環境4の0:00:09）を上回った。
  小〜中規模系ではGPUへの転送オーバーヘッドが相対的に重く、多コアCPUが有利になる。
- ポテンシャル別の目安: EAM/LJ等の軽い力場は多コアCPUが強い。MEAMはGPU/OMP版が
  存在せず純MPI一択。ReaxFFはomp版（CPU）とKokkos版（GPU）の比較が必要。
  実運用の判断は実際の系でベンチを取ってから。

---

## 6. 可視化

[OVITO](https://www.ovito.org/) Basic（無償版）をインストールし、`dump.XX` ファイルを開く。

---

## 検証済み環境一覧

| ホスト | ハードウェア | OS / ツールチェーン | LAMMPS | 構成 | 確認日 |
|---|---|---|---|---|---|
| Xeon 6952Pワークステーション | Xeon 6952P（96C） | Ubuntu 24.04.4 / oneAPI 2025.3.2 | 22Jul2025 u5 | A（lmp_cpu） | 2026/08/29 |
| w7-2495Xワークステーション | w7-2495X + RTX 6000 Ada | Ubuntu 24.04 / oneAPI 2025.1.0 / CUDA 13.1 | — | C（lmp_kokkos） | 2025 |
| RTX 6000 Adaサーバー | Xeon（24C）+ RTX 6000 Ada | Ubuntu 22.04.5 / oneAPI / CUDA 12.6 | 22Jul2025 u4 | C（lmp_kokkos） | 2026/08/31 |
| WSL2各機 | RTX 3060 / A4000 / 4090 | WSL2 Ubuntu / CUDA on WSL | 23Jun2022ほか | B（lmp_gpu） | 2022–2025 |
| プラズマシミュレータ | MI300A | rocm 6.3.3 / openmpi 5.0.7 | — | D・E | 2025 |

## 変更履歴

- 2026/08/31: oneAPIビルドの実行時依存（setvars必須）と環境スクリプト方針の注意書きを追加、
  GPU_ARCHの参照先を公式一覧に変更、Kokkos構成にアーキテクチャ指定と`which mpicxx`の注意を追記、
  検証済み環境一覧にRTX 6000 Adaサーバー（Ubuntu 22.04.5 / CUDA 12.6 / 22Jul2025 u4）を追加。
- 2026/08/29: 全面改訂。表題を統合版に変更、CPU専用サーバー構成（A）と
  そのベンチマーク（環境6）を追加、安定版を22Jul2025系に更新、実行コマンドを環境別に整理。
