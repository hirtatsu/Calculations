## CUDA on WSLを使ってGPGPUで計算する

3steps. Versions should be supported according to the [Support Matrix](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html?ncid=em-prod-337416).
1. NVIDIA Driver installation
2. CUDA Toolkit installation. [Version archives](https://developer.nvidia.com/cuda-toolkit-archive)
3. CuDNN installation



### wslをUPDATEしておく。Powershell開いて以下入力する。
```
wsl --update
```


### 事前準備
```
sudo apt update
sudo apt upgrade -y
```

### 1. NVIDIA Driver for GPU SupportをダウンロードしてWindowsにインストールする
[https://www.nvidia.com/Download/index.aspx?lang=en-us](https://www.nvidia.com/Download/index.aspx?lang=en-us)


### 2. CUDA Toolkits on WSL Ubuntu
[公式マニュアル](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#abstract)

- 古いGPGキーを削除。
```
sudo apt-key del 7fa2af80
```
- CUDA Toolkit (WSL-Ubuntu Package)をインストールする。ここではCuda Toolkit 12.1.1を例に。
```
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.1.1/local_installers/cuda-repo-wsl-ubuntu-12-1-local_12.1.1-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-12-1-local_12.1.1-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-12-1-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt update
sudo apt -y install cuda
```

- PATHを通す
```
cd
vim .bashrc
```
-一番最後の行に以下を追加
```
export PATH=/usr/local/cuda-12.3/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-12.3/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
-追記したら反映させる
```
source .bashrc
```

- インストールされたか確認
```
nvidia-smi
nvcc --version
```


### WSLのUbuntu上でCuDNNを準備する
インストール方法は公式マニュアル見てなんとかする。

[https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)


[https://developer.nvidia.com/rdp/cudnn-archive](https://developer.nvidia.com/rdp/cudnn-archive)
