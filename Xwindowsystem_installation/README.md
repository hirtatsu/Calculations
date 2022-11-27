## WSL上あるいはサーバー上のGUIアプリを使えるようにする
WSLや研究室の計算サーバ上で動くGUIアプリを、手元のPCで表示できるように設定します。

### VcXsrv Windows X ServerをWindows上にインストール
- 以下からVcXsrvをダウンロードしてインストールする
[https://sourceforge.net/projects/vcxsrv/](https://sourceforge.net/projects/vcxsrv/)

- XLaunchを起動する (デスクトップ？にできたはずのXLaunchから起動)

デフォルトのMultiple windowsを選ぶ

<img width="379" alt="xlaunch1" src="https://user-images.githubusercontent.com/64639043/204116530-96f97bdd-495c-4a7d-86c2-529dcbd169db.png">

デフォルトのStart no clientを選ぶ

<img width="378" alt="xlaunch2" src="https://user-images.githubusercontent.com/64639043/204116532-ff978a62-4c26-4c52-bb38-c3a61850a5bf.png">

Extra settings。Disable access controlを有効にするのを忘れずに。

<img width="379" alt="xlaunch3" src="https://user-images.githubusercontent.com/64639043/204116534-5348c899-7bac-4cd8-8242-92b09f76ced7.png">

→ 完了

毎回XLaunchを起動するのが面倒なので、Windowsのスタートアップに登録しておくと便利。

### WSL上で設定する
- x11-appsをインストールする
```
sudo apt install x11-apps
```
- IPアドレスを確認する
```
hostname | xargs dig +short
```

- 以下のように表示される
```
172.23.240.1
192.168.1.33
```
- 例えば192の方を使うなら以下を.bashrcファイルの最終行に追記して保存する
```
export DISPLAY=`hostname | xargs dig +short | grep 192.168.1`:0.0
```
- .bashrcの内容を反映させる
```
source .bashrc
```
- 動作確認。マウスに連動する目玉が出てきたら成功
```
xeyes
```
<img width="133" alt="xlaunch4" src="https://user-images.githubusercontent.com/64639043/204118509-92d7c6c8-0a77-45ad-9989-8eff1024dccf.png">



