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

- (memo) Docker上のUbuntuなどグラフィック環境がない場合は[こちらの方法]([../Docker/Docker_tutorial3)を参照してIntel oneAPIをインストールする。

- WSL上の場合は以下でいけるはず。

```
cd
```

Windows上で[Intel oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.d1jvm6)にアクセスして、Intel oneAPI Base ToolkitとIntel oneAPI HPC Toolkitを以下の通りインストールする。
Linuxにて。Offlineインストールがおすすめ。

### Intel OneAPIの設定を反映する（毎回、要実施）
```
source /opt/intel/oneapi/setvars.sh
```

### （参考）Intel oneapiで入手したコンパイラのPATHを通す
そうすると、毎回上述のコマンドを打つ必要がなくなる。
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
icx -v
```
