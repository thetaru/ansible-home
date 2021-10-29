## GitHub Actions Self-Hosted Runner
### ホスト名
```
$ hostnamectl set-hostname <ホスト名>
```
  
### ネットワーク
```
### IPアドレスの設定
$ nmcli connection modify <コネクション名> ipv4.method manual ipv4.addresses <IPアドレス>/サブネットマスク>

### ゲートウェイの設定
$ nmcli connection modify <コネクション名> ipv4.gateway <IPアドレス>

### DNSの設定
$ nmcli connection modify <コネクション名> ipv4.dns <IPアドレス>

### 自動接続の有効化
$ nmcli connection modify <コネクション名> connection.autoconnect yes

### 設定の有効化
$ nmcli connection up <コネクション名>
```
※ ここで設定したDNSは構築用の一時的な設定です

### 時刻同期
GitHubの認証時に時刻ずれがあると失敗してしまいます。
```
$ systemctl enable --now chronyd.service
```
### GitHub Actions Self-hosted Runner
以下、`GitHub Actions Self-hosted Runner`を`GHASR`と略します。
```
### GHASR用ユーザの作成(このユーザはansible実行ユーザでもあります)
$ useradd -k /dev/null -s /sbin/nologin -d /opt/actions-runner -m gha-runner

### パスワードの設定
$ passwd gha-runner

$ cd /opt/actions-runner

### Settings > Actions > Runners > Create self-hosted runner より 最新のパッケージを確認する
### パッケージをダウンロードして解凍
$ curl -o actions-runner-linux-<arch>-<version>.tar.gz -L https://github.com/actions/runner/releases/download/<version>/actions-runner-linux-<arch>-<version>.tar.gz
$ tar xzf ./actions-runner-linux-x64-<version>.tar.gz

### パッケージの所有者/所有グループを変更
$ chown -R gha-runner:gha-runner /opt/actions-runner

### Settings > Actions > Runners > Create self-hosted runner より トークンを確認する
### パッケージのインストール
$ su -s /bin/bash - gha-runner -c "./config.sh --url https://github.com/thetaru/ansible-home --token XXXX...XXXX"

### GHASRサービスを登録
$ cat <<EOF > /etc/systemd/system/github-actions-runner.service
[Unit]
Description=GitHub Actions Runner
After=network.target

[Service]
WorkingDirectory=/opt/actions-runner
User=gha-runner
Group=gha-runner
ExecStart=/opt/actions-runner/bin/runsvc.sh
Restart=always
KillMode=process
KillSignal=SIGTERM
TimeoutStopSec=5min

[Install]
WantedBy=multi-user.target
EOF

### GHASRサービスを有効化
$ systemctl daemon-reload
$ systemctl enable --now github-actions-runner.service
```
### ユーザ
gha-runnerユーザがrootへ昇格できるようにします。
```
$ vi /etc/pam.d/su
-  #auth            required        pam_wheel.so use_uid
+  auth            required        pam_wheel.so use_uid

### gha-runnerをwheelグループに追加
$ usermod -aG wheel gha-runner
```
### パッケージ
```
### ansibleをインストール
$ yum install epel-release
$ yum install ansible
```
### 設定の反映
```
### システムの再起動
$ systemctl reboot
```
