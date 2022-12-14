## 計算の実行
### ディレクトリの作成とインプットファイルの準備
- openmx3.9/に移動して、計算用の親ディレクトリ(ここではworkdir)を作成する。
```
mkdir workdir
cd workdir
```
- さっそく、test01を計算することにする。test01用の子ディレクトリ作って移動する。
```
mkdir test01
cd test01
```
- ここに、①in.dat(インプットファイル), ②openmx(コンパイルした実行ファイル)を格納する。

### 計算を実行する
- 以下のコマンドを実行する。
```
mpirun -np 8 ./openmx in.dat -nt 2 </dev/null>& log.txt &
# 8, 2: 並列数。環境に応じて適宜変更。
```
- ジョブの状況確認は以下。
```
jobs # 状況確認
bg %XXXX # jobsで確認した停止中のジョブ番号XXXXを再開する
kill %XXXX # jobsで確認した実行中のジョブ番号XXXXを停止する
```
- 計算中のログファイルの状況確認は以下。元に戻るときはctrl+c。
```
tail -F log.txt
```

### 計算結果を確認する
- ログファイルを見る。初めから順番に見るときは以下。Enter押して下に進む。もとに戻るときはq。
```
less log.txt
```
- ログファイルを一括して表示する。
```
cat log.txt
```
- ログファイルの中から、指定の文字列を含む行を表示する。
```
cat log.txt | grep 'XXXXX'
```
- ログファイルの中から、計算にかかった時間を表示する。
```
cat log.txt | grep 'Total Computational Time'
```
- ログファイルの中から、計算で得られた系全体のエネルギーを表示する。
```
cat log.txt | grep 'Utot  ' # 単位は[Hartree]
```
- AIScope(可視化ソフト)用のファイル変換
```
sed -e 's/Cell_Vectors=/\nBOX/g' result.md > result.md3
```
- 出力されたファイルの内、当面重要なのは以下。

      result.md                各MDステップにおける幾何構造 
      result.md2               最終MDステップにおける幾何構造  
      result.md3               各MDステップにおける幾何構造(AIScope用)
      result.tden.cube         Gaussian cube形式の全電子密度

      result.cif               Material Stuido用の初期構造のcifファイル
      result.v0.cube           Gaussian cube形式のKohn-Shamポテンシャル
      result.vhart.cube        Gaussian cube形式のHartreeポテンシャル
      result.dden.cube         原子密度から計算した差電子密度 
      result_rst/              再スタートファイルを保存するディレクトリ

### ssh接続切断後もバックグラウンドで実行続ける方法
- 計算実行コマンドの最初にnohupをつけて、最後に&をつける。例。
```
nohup mpirun -np 8 ./openmx in.dat -nt 2 </dev/null>& log.txt &
```
- ちなみに、実行中に表示されるコマンドラインは自動的にlog.txtに出力される。途中経過を見たい場合は以下。
```
tail -F log.txt
```
