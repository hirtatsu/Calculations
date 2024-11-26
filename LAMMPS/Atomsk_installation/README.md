## Atomskのインストール
MDで計算するモデルを簡単に作成するためのソフトウェア。

公式は[こちら](https://atomsk.univ-lille.fr/tutorial_install.php)

### Ubuntuへのインストール
- 必要なファイルのダウンロード [https://atomsk.univ-lille.fr/dl.php](https://atomsk.univ-lille.fr/dl.php)。Ubuntuの場合は、Debian/Ubuntu(64 bits)を選べばよい。その場合は以下。
```
cd
wget https://atomsk.univ-lille.fr/code/atomsk_b0.13.1_amd64.deb
```
- dpkgして完成。
```
sudo dpkg -i atomsk_b0.13.1_amd64.deb
```

- 動作確認。格子定数4.02ÅでFCC構造のAlの構造ファイル.xsfを作成する。各原子の座標が記載されたXSFファイルが作成される
```
atomsk --create fcc 4.02 Al aluminium.xsf
```

### モデリング
マニュアルは[こちら](https://atomsk.univ-lille.fr/doc/en/index.html)

- ユニットセルを作成する (結晶方位を指定しない場合 (XYZ方向が[100]方向になる))
```
atomsk --create fcc 3.6147 Cu ファイル.xsf
```
- ユニットセルを作成する (結晶方位を指定する場合 (例: XYZ方向にそれぞれ[121], [-101], [1-11]の場合))
```
atomsk --create fcc 3.6147 Cu orient [121] [-101] [1-11] ファイル.xsf
```
- ユニットセルを繰り返してスラブを作る
```
atomsk 元のファイル.xsf -duplicate X方向の繰り返し数 Y方向の繰り返し数 Z方向の繰り返し数 出来上がったファイル.xsf
```

- 指定の領域から原子を削除する
```
atomsk 元のファイル.xsf -select in box X方向の下限 Y方向の下限 Z方向の下限 X方向の上限 Y方向の上限 Z方向の上限 -remove-atom select 出来上がったファイル.xsf
```
X,Y,Z方向の上下限には、① INFか-INF、② 端からの距離[Å]、または、③ Boxに対する比率(例: 0.1*box)が使える。

- 原子の位置をシフトさせる
```
atomsk 元のファイル.xsf -shift X方向のシフト量 Y方向のシフト量 Z方向のシフト量 出来上がったファイル.xsf
```

- 二つのモデルを結合する
```
atomsk --merge Z 2 一つ目のファイル.xsf 二つ目のファイル.xsf 出来上がったファイル.xsf
```
  --mergeの後に結合する方向(X,Y,Z)と結合する数を指定する

- BOXを原子配列にフィットするように再定義する
```
atomsk 元のファイル -rebox 出来上がったファイル.xsf
```

- 設定ファイルに基づいてモデルファイルを変換する
xxx.txtを用意する。内容は例えば以下の通り。詳細は[こちら](https://atomsk.univ-lille.fr/doc/en/option_properties.html)。
```
# 全ての原子の位置をZ方向に10オングストローム動かす
displacement function
ux = 0
uy = 0
uz = 10

# シフトした後にZ方向にスーパーセルを指定した寸法に変更する
conventional
# XYZサイズ
100.0 100.0 100.0
# 各ベクトルの成す角
90.0 90.0 90.0
```
その後に以下を実行する
```
atomsk 元のファイル名 -properties xxx.txt 出来上がったファイル名
```
