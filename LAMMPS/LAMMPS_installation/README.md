## インストール準備
### 必要なパッケージを準備する
```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential cmake -y
```

---

## 前準備 (計算サーバにはすでにIntel oneAPIを導入済みのためスキップ)
### (1) Intel OneAPIを使う場合
### インストールで必要なパッケージをIntel oneAPI Toolkitsで入手する

- Docker上のUbuntuなどグラフィック環境がない場合は[こちらの方法](../Docker_tutorial3/README.md)を参照してIntel oneAPIをインストールし、LAMMPSのインストールへ進む。

- WSL上の場合は以下でいけるはず。

```
cd
```

Windows上で[Intel oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.d1jvm6)にアクセスして、Intel oneAPI Base ToolkitとIntel oneAPI HPC Toolkitを以下の通りインストールする。

まず、[Intel oneAPI Base Toolkitのダウンロード](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html)をクリック。

- Operating system: Linux
- Select distribution: Online

を選択して、表示される'Command Line Download'に記載のコードをUbuntu上で実行する。
すると、インストール画面が別ウインドウで立ち上がるので、画面の指示に従ってインストールする。

続いて、[Intel oneAPI HPC Toolkitのダウンロード](https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit-download.html)をクリック。上と同様にインストールする。
カスタムインストールにて、DPC++/C++ Compiler, MPI Library, Fortran Compilerをインストールする。

### Intel oneapiで入手したコンパイラのPATHを通す
```
cd
vim .bashrc
```
で開いて、最後に行を追加して以下を入力して保存する。
```
source /opt/intel/oneapi/setvars.sh
```
そして、以下のコマンドでPATHを反映させる。
```
source .bashrc
```
ちゃんとインストールできたか確認する。バージョンとか表示されればOK。
```
icc -v
```

### (2) GNUコンパイラを使う場合
### 必要なパッケージをインストールする
```
sudo apt install -y cmake build-essential ccache gfortran openmpi-bin libopenmpi-dev \
                    libfftw3-dev libjpeg-dev libpng-dev python3-dev python3-pip \
                    python3-virtualenv libblas-dev liblapack-dev libhdf5-serial-dev \
                    hdf5-tools clang-format ffmpeg
```

---

## 前準備2 (VORONOIパッケージを使う場合)
- [VORO++](https://math.lbl.gov/voro++/)をインストールする。上記のダウンロードページから、最新バージョンのダウンロードリンクをコピーしてくる。
```
cd
wget https://math.lbl.gov/voro++/download/dir/voro++-0.4.6.tar.gz
tar -xvf voro++-0.4.6.tar.gz # 2023.6.10現在、Ver. 4.6が最新
cd voro++-0.4.6
make -j
sudo make install
```
- PATHを通す。デフォルトでは/usr/local/binになっているはず。.bashrcの一番下あたりに以下を記入してから再起動。
```
export PATH=/usr/local/bin:$PATH
```



## LAMMPSのインストール

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
### cmakeでMakefileを作成する(MPI, MANYBODY, VORONOI, ReaxFFパッケージを追加)
- Intel OneAPIを用いる場合
```
cmake ../cmake/presets/basic.cmake -DCMAKE_C_COMPILER=icc -DCMAKE_CXX_COMPILER=icpc -DCMAKE_Fortran_COMPILER=ifort -D BUILD_MPI=yes -D PKG_MANYBODY=yes -D PKG_VORONOI=yes -D BUILD_REAXFF=yes ../cmake
```
- GNUコンパイラを用いる場合
```
cmake ../cmake/presets/basic.cmake -D BUILD_MPI=yes -D PKG_MANYBODY=yes ../cmake
```
コンパイルして生成されるファイル名は通常「lmp」。もし「lmp_XXX」にしたい場合は、上記cmakeするときに、以下を追加しておくこと
```
-D LAMMPS_MACHINE=xxx
```
にする。

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
最終行に以下を追加する
```
export PATH=~/.local/bin:$PATH
```
以下のコマンドで反映させる。
```
cd
source .bashrc
```

### 最後に確認
- 以下でエラーがでなければ成功
```
lmp
```

## お試し計算
### インプットファイル編集と実行 
もともと用意されている計算例(melt)を実行してみる。まずInputファイルを編集。
```
cd
cd MD/lammps-stable_3Mar2020/examples/melt
vim in.melt
```
インプットファイル(in.melt)をvimで以下の通り編集する。

編集するのは22行目。行頭の♯を削除するだけ。
```
dump            id all atom 50 dump.melt
```

そして、コマンドライン上で実行する。

シングルコアで計算する場合
```
lmp < in.melt
```
マルチコア(例: 4コア)で計算する場合
```
mpirun -np 4 lmp < in.melt
```
マルチコアでバックグラウンドで計算する場合
```
nohup mpirun -np 4 lmp < in.lmp &
```

なんか計算がはじまって、最後に'Total wall time: xx:xx:xx'と表示されていれば成功！

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

### 可視化
Windowsに可視化ソフトをインストールする。OVITO Basic (Proは有料)。
https://www.ovito.org/


dump.XXファイルをOVITOで開けば可視化できます。
