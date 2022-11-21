## 準備
### (1) Intel OneAPIを使う場合
### インストールで必要なパッケージをIntel oneAPI Toolkitsで入手する
```
cd
```

Windows上で[Intel oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.d1jvm6)にアクセスして、Intel oneAPI Base ToolkitとIntel oneAPI HPC Toolkitを以下の通りインストールする。

まず、[Intel oneAPI Base Toolkitのダウンロード](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html)をクリック。

- Operating system: Linux
- Select distribution: Online

を選択して、表示される'Command Line Download'に記載のコードをUbuntu上で実行する。
すると、インストール画面が別ウインドウで立ち上がるので、画面の指示に従ってインストールする。
(カスタムインストールを選択する場合は、Math Carnel Libraryを必ずインストールすること)

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
### cmakeでビルドする(MPI, MANYBODYパッケージを追加)
```
cmake ../cmake/presets/basic.cmake -DCMAKE_C_COMPILER=icc -DCMAKE_CXX_COMPILER=icpc -DCMAKE_Fortran_COMPILER=ifort -D BUILD_MPI=yes -D PKG_MANYBODY=yes ../cmake
```
### コンパイルする
```
make -j 4  # Numberは並列コア数
```
```
make install
```

### 最後に確認
```
lmp
```
と入力して、エラーがでなければインストール成功。

### (もしPathが通ってなければ)Pathを通す
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



### (2) GNUコンパイラを使う場合
### 必要なパッケージをインストールする
```
sudo apt install -y cmake build-essential ccache gfortran openmpi-bin libopenmpi-dev \
                    libfftw3-dev libjpeg-dev libpng-dev python3-dev python3-pip \
                    python3-virtualenv libblas-dev liblapack-dev libhdf5-serial-dev \
                    hdf5-tools
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
### cmakeでビルドする(MPI, MANYBODYパッケージを追加)
```
cmake ../cmake/presets/basic.cmake -D BUILD_MPI=yes -D PKG_MANYBODY=yes ../cmake
```
### コンパイルする
```
make -j 4  # Numberは並列コア数
```
```
make install
```

### 最後に確認
```
lmp
```
と入力して、エラーがでなければインストール成功。

### (もしPathが通ってなければ)Pathを通す
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
