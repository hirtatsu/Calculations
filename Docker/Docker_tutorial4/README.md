## DockerでCudaを使える環境を構築する
CudaはNVIDIAが提供するGPUを用いた計算(GPGPU)に用いるツールです。

### NVIDIAドライバのインストール (① WSL環境の場合)
[https://www.nvidia.com/Download/index.aspx?lang=en-us](https://www.nvidia.com/Download/index.aspx?lang=en-us)

### NVIDIAドライバのインストール (② Linux OS (Ubuntu)機の場合)
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
recommendedを見つける

- ドライバのインストール
```
sudo apt install 「例: nvidia-driver-450」 -y
```
(sudo ubuntu-drivers autoinstallでもいけるらしい)

- 再起動
```
sudo reboot
```

### NVIDIA Container Toolkitの準備
- インストールする
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt update && sudo apt install -y nvidia-container-toolkit
```
- Docker Engineのリスタート
- Linux機の場合
```
sudo systemctl restart docker
```
- WSL環境の場合
```
sudo service docker restart
```
- 確認
```
nvidia-container-cli info
```

### DockerでGPUを使う
続きは[ここ](https://qiita.com/tatsuya11bbs/items/3af03e704812b6c89965)
