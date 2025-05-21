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

まず、[Intel oneAPI Base Toolkitのダウンロード](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html)をクリック。

- Operating system: Linux
- Select distribution: Online

を選択して、表示される'Command Line Download'に記載のコードをUbuntu上で実行する。
すると、インストール画面が別ウインドウで立ち上がるので、画面の指示に従ってインストールする。
※カスタムを選ぶ場合、必ずMath Karnel Libraryを選ぶこと。

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
