## CUDA on WSLを使ってGPGPUでLAMMPS計算する
### GPGPU環境を構築する
- [こちら](../../GPGPU/README.md)
- [公式マニュアル](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#abstract)


### その他の必要なパッケージを準備する
```
sudo apt install -y cmake build-essential ccache gfortran openmpi-bin libopenmpi-dev \
                    libfftw3-dev libjpeg-dev libpng-dev python3-dev python3-pip \
                    python3-virtualenv libblas-dev liblapack-dev libhdf5-serial-dev \
                    hdf5-tools ffmpeg
```                    
### MD用ディレクトリを作成して移動
```
cd
mkdir MD
cd MD
```

### LAMMPSをダウンロード、解凍して当該ディレクトリへ移動
```
wget https://download.lammps.org/tars/lammps-stable.tar.gz
tar xvzf lammps-stable.tar.gz
cd lammps-23Jun2022
```

### buildディレクトリを作成して移動
```
mkdir build
cd build
```
### cmakeでMakefileを作成する(GPU関連, MPI, MANYBODY, VORONOI, ReaxFFパッケージを追加, Intelコンパイラを使用)
- GPU_ARCHは[こちら](https://qiita.com/k_ikasumipowder/items/1142dadba01b42ac6012)でチェック。例: GeForce RTX 3060、RTX A4000はsm_86、RTX4090はsm_89、6000Adaもsm_89。
```
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
- Ubuntuのバージョン次第では、nvccとintelコンパイラが競合して(?)うまくいかないかも？その場合は以下でどうでしょう。
```
cmake -C ../cmake/presets/most.cmake \
 -D LAMMPS_MACHINE=gpu \
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
### cmakeでMakefileを作成する(Kokkos＋CUDA用, Intelコンパイラを使用)
- 詳細は[こちら](https://docs.lammps.org/Build_extras.html#kokkos)でチェック。
```
cmake -C ../cmake/presets/most.cmake \
 -D LAMMPS_MACHINE=kokkos \
 -D CMAKE_C_COMPILER=icx \
 -D CMAKE_CXX_COMPILER=icpx \
 -D CMAKE_Fortran_COMPILER=ifx \
 -D FFT=MKL \
 -D BUILD_MPI=yes \
 -D BUILD_OMP=yes \
 -D PKG_KOKKOS=yes \
 -D Kokkos_ENABLE_CUDA=yes \
 -D Kokkos_ENABLE_OPENMP=yes \
 -D Kokkos_ARCH_ADA89=yes \
 -D Kokkos_ARCH_NATIVE=yes \
 -D PKG_MEAM=yes \
 -D PKG_MANYBODY=yes \
 -D PKG_VORONOI=yes \
 -D DOWNLOAD_VORO=yes \
 -D PKG_REAXFF=yes \
 ../cmake
```
### AMDのAPUアクセラレータ（AMD Instinct MI300A）を用いる場合（Plasma Simulator）
```
module load openmpi/5.0.7/rocm6.3.3
```
```
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
-D PKG_REAXFF=yes ../cmake
```

CMAKEのプリセットは[こちら](https://docs.lammps.org/Build_package.html#cmake-presets-for-installing-many-packages)

### ビルドする
```
make -j 4  # Numberは並列コア数
```
```
make install
```

### Pathを通す
```
cd
vim .bashrc
```
最終行に以下を追加する(Buildファイルの場所が/home/「ユーザ名」/.local/binの場合)
```
export PATH=/home/「ユーザ名」/.local/bin:$PATH
```
以下のコマンドで反映させる。
```
cd
source .bashrc
```

### 最後に確認
```
lmp_gpu
```
と入力して、エラーがでなければインストール成功。Ctrl+cで終了。

### お試し計算
もともと用意されている計算例(melt)を実行してみる。まずInputファイルを編集。
```
cd
cd MD/lammps-stable_23Jun2022/examples/melt
vim in.melt
```
インプットファイル(in.melt)をvimで以下の通り編集する。

編集するのは22行目。行頭の♯を削除するだけ。
```
dump            id all atom 50 dump.melt
```

そして、コマンドライン上で実行する。

- GPUで計算する場合
```
lmp_gpu -sf gpu -pk gpu 1 -in in.melt
```
なんか計算がはじまって、最後に'Total wall time: xx:xx:xx'と表示されていれば成功！

- GPUとCPUのハイブリッドで計算する場合
```
mpirun -n 8 lmp_gpu -sf gpu -pk gpu 1 -in in.melt
```

### 結果の確認
```
ls
```
すると、
```
dump.melt # ← 各時間ステップにおける原子の配置に関するデータ。後で可視化する。
log.lammps # ← ログファイル。
```
ができているはず。

### 計算時間の比較
計算内容: examples/melt/in.meltをもとに、「region box block 0 50 0 50 0 50」「run 2500」に変更。

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

計算環境5: GPUにAMDAMD Instinct MI300A @プラズマシミュレータ。

- GPUの場合: 0:00:24 (-93%)

### 可視化
Windowsに可視化ソフトをインストールする。OVITO Basicを選んでインストール (Proは有料)。
https://www.ovito.org/


dump.XXファイルをOVITOで開けば可視化できます。
