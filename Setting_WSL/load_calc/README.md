## 計算環境をスイッチで導入できるように設定する
### 環境読み込みスクリプト(.research_env)をGithub Gistからダウンロードする
```
wget https://gist.github.com/hirtatsu/171b170489c3cc35e7b6f206b416bbe5/raw/d04beac918102128e7cf1b569d1183bfabd21cf0/.research_env
```

### .bashrcに追記する
```
# ======================================================================
# Tatsumi Research Environment Config (Smart Auto-Detect Edition)
# ======================================================================
# .bashrc (または .zshrc) の一番下に、以下の1行を追加する。
[ -f ~/.research_env ] && source ~/.research_env
# ======================================================================
```

