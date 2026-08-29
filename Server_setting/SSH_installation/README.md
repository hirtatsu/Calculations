# 計算サーバのSSH設定

最終更新: 2026-08 ／ 対象: Ubuntu 22.04 / 24.04 / 26.04

研究室の計算サーバに、手元のPCからSSHで接続できるようにするための**サーバ側**の設定です。接続する側（クライアント）の設定は[計算サーバへの接続方法](../Howtoaccess_server/README.md)を参照してください。

---

## 1. SSHサーバのインストール

```
sudo apt update
sudo apt install openssh-server
```

起動状態を確認します。

```
sudo systemctl status ssh
```

`active` と表示されていればOKです。

### 操作一覧

| 操作 | コマンド |
|---|---|
| 停止 | `sudo systemctl stop ssh` |
| 起動 | `sudo systemctl start ssh` |
| 再起動 | `sudo systemctl restart ssh` |
| 設定のリロード | `sudo systemctl reload ssh` |
| 自動起動の有効化 | `sudo systemctl enable ssh` |
| 自動起動の無効化 | `sudo systemctl disable ssh` |

---

## 2. セキュリティ設定

**インストールしただけの状態はパスワード認証が有効です。** 計算サーバを学内ネットワークや外部から接続できる状態に置く場合、パスワード認証は総当たり攻撃の対象になります。公開鍵認証に切り替えてください。

### 手順の順序が重要です

締め出しを防ぐため、必ずこの順序で進めてください。

1. 利用者の公開鍵をサーバに登録する
2. 公開鍵でログインできることを確認する
3. パスワード認証を無効化する
4. **現在のセッションを閉じずに**、別の端末から接続できることを確認する

3の後に接続できなくなると、サーバのコンソールに直接触れるまで復旧できません。

### 公開鍵の登録

利用者側で鍵を作成し（[接続方法](../Howtoaccess_server/README.md)を参照）、公開鍵をサーバに登録します。利用者自身が手元から実行する場合は以下が簡単です。

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub ユーザ名@ホスト名
```

サーバ側で手作業する場合は、対象ユーザのホームディレクトリに配置します。

```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys     # 公開鍵の内容を貼り付ける
chmod 600 ~/.ssh/authorized_keys
```

> パーミッションが緩いと公開鍵認証は動作しません。`.ssh` は700、`authorized_keys` は600にしてください。

### sshdの設定

Ubuntuでは `/etc/ssh/sshd_config` に加えて、`/etc/ssh/sshd_config.d/` 以下の `.conf` ファイルも読み込まれます。**後から読み込まれた設定が優先されるため、本体を編集しても効かないことがあります。**

まず、既存の設定を確認します。

```
sudo ls /etc/ssh/sshd_config.d/
```

`50-cloud-init.conf` などが存在し、その中で `PasswordAuthentication yes` が指定されている場合があります。この場合、本体の `sshd_config` を変更しても上書きされます。

新しい設定ファイルを作り、確実に最後に読み込まれるようにします。

```
sudo vim /etc/ssh/sshd_config.d/99-hardening.conf
```

以下を記述します。

```
# 公開鍵認証を使う
PubkeyAuthentication yes

# パスワード認証を無効にする
PasswordAuthentication no
KbdInteractiveAuthentication no

# rootでの直接ログインを禁止する
PermitRootLogin no
```

設定に文法エラーがないか確認します。

```
sudo sshd -t
```

何も表示されなければ問題ありません。実際に適用される設定値を確認します。

```
sudo sshd -T | grep -E "passwordauthentication|permitrootlogin|pubkeyauthentication"
```

意図した値になっていれば、設定を反映します。

```
sudo systemctl reload ssh
```

**この時点で現在のセッションは閉じずに、別の端末から接続できることを確認してください。**

### ポート番号を変更する場合

Ubuntu 22.10以降、SSHはソケット起動（socket activation）に変更されています。**`sshd_config` の `Port` を書き換えても反映されません。** 変更する場合は `ssh.socket` 側を編集します。

```
sudo systemctl edit ssh.socket
```

```
[Socket]
ListenStream=
ListenStream=2222        # 任意のポート番号
```

```
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
```

ポート変更は攻撃ログを減らす効果はありますが、それ自体はセキュリティ対策ではありません。公開鍵認証への移行を優先してください。

---

## 3. Tailscaleの設定

Tailscaleを使うと、グローバルIPやポート開放なしに、外部から計算サーバへ接続できます。研究室のネットワーク構成やファイアウォールの制約でSSHポートを開けられない場合に有効です。

### インストール

```
curl -fsSL https://tailscale.com/install.sh | sh
```

起動して機器を登録します。

```
sudo tailscale up
```

表示されたURLをブラウザで開き、アカウントにログインして機器を登録します。

### 状態の確認

```
sudo systemctl status tailscaled
```

`active` と表示されていればOKです。割り当てられたIPアドレスは以下で確認できます。

```
tailscale ip -4
tailscale status
```

このIPアドレスに対して、通常どおりSSH接続できます。

### 操作一覧

| 操作 | コマンド |
|---|---|
| 停止 | `sudo systemctl stop tailscaled` |
| 起動 | `sudo systemctl start tailscaled` |
| 再起動 | `sudo systemctl restart tailscaled` |
| 自動起動の有効化 | `sudo systemctl enable tailscaled` |
| 自動起動の無効化 | `sudo systemctl disable tailscaled` |
| ネットワークから切断 | `sudo tailscale down` |

> `tailscaled` は `systemctl reload` に対応していません。設定を変えた場合は `restart` を使ってください。

---

## 関連ページ

- [計算サーバへの接続方法](../Howtoaccess_server/README.md) — クライアント側の設定
- [Linux作業Tips](../../Others/Tips/README.md) — ユーザ管理、ストレージの増設
