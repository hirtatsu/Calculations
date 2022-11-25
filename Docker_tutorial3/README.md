## Interl OneAPIのインストール
Docker上のUbuntuでは(Graphical環境を準備しない限り)コマンドでインストールする必要がある。

### 準備
```
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
sudo apt update
```

### Intel oneAPI Base Toolkitのインストール
```
sudo apt install intel-basekit -y
```

### Intel oneAPI HPC Toolkitのインストール
```
sudo apt install intel-hpckit -y
```
### Pathを通す
- .bashrcを編集する
```
cd
vim .bashrc
```
- 最後の行に以下を追記する
```
source /opt/intel/oneapi/setvars.sh
```
- 反映させる
```
source .bashrc
```
- インストールされているか確認する。Version情報が表示されたらOK。
```
icc -v
```
