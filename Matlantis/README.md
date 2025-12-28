# Matlantis
## 初期環境構築
### .bashrcを編集
```
cd
vim .bashrc
```
以下を末尾に追記。
```
use_venv python313 # pyenvを読み込むように設定。環境を抜けるにはdeactivate_venv
alias ls='ls --color=auto -ltr' # lsの表示を好みに変更
```
