# Matlantis
## 初期環境構築 (Matlantisのターミナル上で作業することに）
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
以下で反映。
```
source .bashrc
```

### Atomskのインストール


### ssh接続環境の構築
Matlantis公式ドキュメント参照：[SSH接続 設定方法](https://matlantis.zendesk.com/hc/ja/articles/40524152374169-SSH%E6%8E%A5%E7%B6%9A-%E8%A8%AD%E5%AE%9A%E6%96%B9%E6%B3%95)

### LAMMPSのインストール（PFPを力場としたLAMMPS計算）
Matlantis公式ドキュメント参照：[LAMMPS連携パッケージ(matlantis-lammps)のインストール](https://matlantis.zendesk.com/hc/ja/articles/33994518415257-LAMMPS%E9%80%A3%E6%90%BA%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8-matlantis-lammps-%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E6%96%B9%E6%B3%95)
