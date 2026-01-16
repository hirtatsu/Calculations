## 計算環境スイッチを設置
ワンタッチでIntel OneAPIとCuda環境を読み込む

### 計算環境読み込みスクリプトのダウンロードと設定
- Github Gistからダウンロードする（.research_env）
```
wget https://gist.github.com/hirtatsu/171b170489c3cc35e7b6f206b416bbe5/raw/d04beac918102128e7cf1b569d1183bfabd21cf0/.research_env
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

### 計算環境の読み込み方法
- 以下のコマンドを実施する
```
load_calc
```
- ターミナルに`[HPC-MODE]`と表示されるようになる。
