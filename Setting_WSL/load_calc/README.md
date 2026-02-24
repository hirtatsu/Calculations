## 計算環境スイッチを設置
ワンタッチでIntel OneAPIとCuda環境を読み込む

### 計算環境読み込みスクリプトのダウンロードと設定
- Github Gistからダウンロードする（.research_env）
```
https://gist.githubusercontent.com/hirtatsu/171b170489c3cc35e7b6f206b416bbe5/raw/a1e81452c1c36ac1b8a4156334c3ddea0a2417de/.research_env
```
- .bashrcに追記する
```
# ======================================================================
# Tatsumi Research Environment Config (Smart Auto-Detect Edition)
# ======================================================================
# .bashrc (または .zshrc) の一番下に、以下の1行を追加する。
[ -f ~/.research_env ] && source ~/.research_env
# ======================================================================
```

## 計算(HPC)環境の読み込み方法
### 動作確認
- 以下のコマンドを実施する
```
load_calc
```
- ターミナルに`[HPC-MODE]`と表示されるようになる。例。
```
XXXXXXXXXXXXXXXXX:~$ load_calc
Loading HPC Environment...
  -> Intel OneAPI Loaded.
  -> System CUDA Loaded: /usr/local/cuda
✅ Ready for HPC calculations (OpenMX / LAMMPS).
```
- Intel OneAPI。バージョン情報等表示されればOK。
```
icx --version
```

- Nvidia Cuda Toolkit。バージョン情報等表示されればOK。
```
nvcc --version
```


## 計算(python @ miniforge)環境の読み込み方法
### 動作確認
- 以下のコマンドを実施する
```
load_py
```
- ターミナルに`[ML-MODE]`と表示されるようになる。例。
```
XXXXXXXXXXXXXXXXX:~$ load_py
✅ Miniforge Environment Activated: test-env
```
- Conda。バージョン情報等表示されればOK。
```
conda --version
```
