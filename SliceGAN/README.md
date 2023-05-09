# SliceGAN
与えた2次元画像から、GAN (Generative Adversarial Network（敵対的生成ネットワーク）)を用いて3次元モデルを構築する。

### 前提
- Python環境が構築されていること

### インストール
- ホームディレクトリにて、Gitからもってくる
```
git clone https://github.com/stke9/SliceGAN.git
```
- SliceGANディレクトリに移動
```
cd SliceGAN/
```
- SliceGANを実行してみる
```
python3 run_slicegan.py
```
- Pythonのパッケージで足りない場合エラーが出るので、適宜Pipでインストールする
```
pip install (足りないパッケージ名)
```
- 以下のメッセージが出れば一旦インストールは完了。
```
run_slicegan.py: error: the following arguments are required: training
```

### run_slicegan.pyの編集


