## DockerでCudaを使える環境を構築する
CudaはNVIDIAが提供するGPUを用いた計算(GPGPU)に用いるツールです。

### WSL環境の場合はWindowsでNVIDIAドライバーをインストールする
[https://www.nvidia.com/Download/index.aspx?lang=en-us](https://www.nvidia.com/Download/index.aspx?lang=en-us)

### Linux OS (Ubuntu)におけるNVIDIAドライバーのインストール
- ドライバーがインストールされているか確認
```
nvidia-smi
```
- リポジトリの追加
```
sudo add-apt-repository ppa:graphics-drivers/ppa
```
- インストール可能なドライバの表示
```
ubuntu-drivers devices
```
- ドライバのインストール
```
sudo apt install nvidia-driver-450
```
- 再起動
```
sudo reboot
```

### NVIDIA Container Toolkitの準備
