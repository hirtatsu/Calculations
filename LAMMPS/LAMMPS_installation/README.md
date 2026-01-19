## インストール準備
### 必要なパッケージを準備する
```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential cmake -y
```

---

## 前準備 (計算サーバにはすでにIntel oneAPIを導入済みのためスキップ)
## Intel oneAPI Toolkitsを入手する
### インストール
Windows上で[Intel oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.d1jvm6)にアクセスして、Intel oneAPI Base ToolkitとIntel oneAPI HPC Toolkitを以下の通りインストールする。
- Operating system: Linux
- Select distribution: Offline
インストール完了する。

### 環境呼び出し。計算モードの読み込みスイッチを未導入であれば以下を実行する。
```
source /opt/intel/oneapi/setvars.sh
```
ちゃんと環境を呼び出せたか確認する。バージョンとか表示されればOK。
```
icx -v
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
### cmakeでMakefileを作成する(MPI, MANYBODY, VORONOI, MEAM, ReaxFFパッケージを追加)
- Intel OneAPIを用いる場合
```
cmake -C ../cmake/presets/most.cmake \
-D LAMMPS_MACHINE=cpu \
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

上記でコンパイルして生成されるファイル名は「lmp_cpu」。もし「lmp_XXX」にしたい場合は、上記cmakeするときに、以下を追加しておくこと
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
