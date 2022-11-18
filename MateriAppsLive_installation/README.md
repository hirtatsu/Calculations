## MateriApps LIVE!のインストール

- はじめてお手軽に材料科学計算を行うのに便利なツール。東大物性研ほかが提供。
- 仮想デスクトップソフトウェア(VirtualBox)上に、計算環境がすでに整ったLinux (Debian)をブートする。
- 計算環境構築で最初の難関、ソフトウェアのコンパイルをせずにすぐ計算を体験できる。

※　M1 Mac上ではVirtualBoxが動かないので、別途用意されているDocker版を使うとよい。

### 提供される計算ソフト
[https://github.com/cmsi/MateriAppsLive/wiki/ApplicationsAndTools](https://github.com/cmsi/MateriAppsLive/wiki/ApplicationsAndTools)


### 配布物一式のダウンロード

- [setup.pdf](https://github.com/cmsi/malive-tutorial/raw/master/setup/setup.pdf)
- [README.html](https://github.com/cmsi/MateriAppsLive/wiki/MateriAppsLive-ova)
- [VirtualBOXインストーラ](https://www.virtualbox.org/wiki/Downloads) VirtualBox XXXX platform packagesを探してダウンロード
- [MateriApps LIVE! VirtualBoxディスクイメージ](https://sourceforge.net/projects/materiappslive/files/) MateriAppsLive-*-amd64.ovaを探してダウンロード

### VirtualBoxのインストール
- Windows版の場合: VirtualBox-*-Win.exe
- Mac版の場合: VirtualBox-*-OSX.dmg
をダブルクリックしてインストールする

### MateriApps LIVE!のインポート
- MateriAppsLive-*.ovaをダブルクリック
- VirtualBoxが起動してインポート画面が出るので、「インポート」を押す
- 数分待って完了するとマネージャーが起動する

### VirtualBoxからの起動とログイン、使い方
- VirtualBoxのマネージャー画面から、画面内のMateriApps...を選択の上、起動ボタンを押す。
- ログイン画面が表示されるまで待つ
- ログイン情報は以下の通り
　　- ユーザ名: user
　　- パスワード: live
- デスクトップ画面が表示されればインストール成功
- ターミナル(スタートメニュー → System Tools → LXTerminal)を開いて各種計算ができる
- 各ソフトの使い方は、MateriAppsの各ソフトのレビューに参考情報が見つかるかも。
- 例えばOpenMXの場合。[https://ma.issp.u-tokyo.ac.jp/app-post-category/review?appid=594](https://ma.issp.u-tokyo.ac.jp/app-post-category/review?appid=594)
