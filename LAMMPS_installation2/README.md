## CUDA on WSLをセッティングしてGPGPUを活用する
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
sudo apt -y install cuda
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
wget https://github.com/lammps/lammps/archive/stable_3Mar2020.tar.gz
tar xvzf stable_3Mar2020.tar.gz
cd lammps-stable_3Mar2020
```

### buildディレクトリを作成して移動
```
mkdir build
cd build
```
### cmakeでビルドする(MPI, MANYBODYパッケージを追加)
```
cmake ../cmake/presets/basic.cmake -D PKG_GPU=yes -D GPU_API=cuda -D GPU_ARCH=SM86 -D BUILD_MPI=yes -D PKG_MANYBODY=yes ../cmake
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
