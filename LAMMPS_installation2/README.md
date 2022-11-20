## CUDA on WSLを使ってGPGPUでLAMMPS計算する
[https://docs.nvidia.com/cuda/wsl-user-guide/index.html#abstract](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#abstract)

### NVIDIA Driver for GPU SupportをダウンロードしてWindowsにインストールする
[https://www.nvidia.com/Download/index.aspx?lang=en-us](https://www.nvidia.com/Download/index.aspx?lang=en-us)

### wslをUPDATEする。Powershell開いて以下入力する。
```
wsl --update
```

### WSLのUbuntu上でCUDA Toolkitを準備する
- 古いGPGキーを削除。
```
sudo apt-key del 7fa2af80
```
- CUDA Toolkit (WSL-Ubuntu Package)をインストールする
```
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda-repo-wsl-ubuntu-11-8-local_11.8.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-11-8-local_11.8.0-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-11-8-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt update
sudo apt upgrade -y
sudo apt install cuda -y
```

- PATHを通す
```
cd
vim .bashrc
```
-一番最後の行に以下を追加
```
export PATH=/usr/local/cuda-11.8/bin:$PATH
```
-追記したら反映させる
```
source .bashrc
```

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
### cmakeでビルドする(GPU関連, MPI, MANYBODYパッケージを追加)
- GPU_ARCHは[こちら](https://qiita.com/k_ikasumipowder/items/1142dadba01b42ac6012)でチェック。例: GeForce RTX 3060、RTX A4000はsm_86。
```
cmake ../cmake/presets/basic.cmake -D PKG_GPU=yes -D GPU_API=cuda -D GPU_ARCH=sm_86 -D BUILD_MPI=yes -D PKG_MANYBODY=yes ../cmake
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

- シングルコアで計算する場合
```
lmp < in.melt
```
- マルチコア(例: 4コア)で計算する場合
```
mpirun -np 4 lmp < in.melt
```

- GPUで計算する場合
```
lmp -sf gpu -pk gpu 1 -in in.melt
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
