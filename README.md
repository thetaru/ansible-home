# ansible-home
## ■ 構成
- 内部権威DNSは疑似hidden master型としている(master: github, slave: dns-01, dns-02)
  - 実体はdns-01とdns-02はmasterでレコードを配られているだけ

- zabbixの構成はAnsibleで管理しているが、HA構成(Pacemaker+Corosync+DRBD)は手動で設定していることに注意する  
- zabbix6.0からHAが公式サポートされたので切り替える
## ■ ホスト
|ホスト名|OS|IPアドレス|label(host)|役割|
|:---|:---|:---|:---|:---|
|dns-01|CentOS 8|192.168.137.1|dns-01|名前解決(内部権威)|
|dns-02|CentOS 8|192.168.137.2|dns-02|名前解決(内部権威)|
|dns-03|CentOS 8|192.168.137.3|dns-03|名前解決(キャッシュ)|
|ntp-01|CentOS 8|192.168.137.5|ntp-01|時刻同期|
|ntp-02|CentOS 8|192.168.137.6|ntp-02|時刻同期|
|log-01|CentOS 8|192.168.137.10|log-01|ログ|
|proxy-01|CentOS 8|192.168.137.20|proxy-01|フォワードプロキシ|
|proxy-02|CentOS 8|192.168.137.21|proxy-02|フォワードプロキシ|
|lb-01|CentOS 8|192.168.137.25|lb-01|ロードバランサ|
|zbx-01|CentOS 8|192.168.137.30|zbx-01|Zabbix(Active)|
|zbx-02|CentOS 8|192.168.137.31|zbx-02|Zabbix(Stanby)|
|mail-01|CentOS 8|192.168.137.35|mail-01|メール中継|
|ad-01|Windows Server 2019|192.168.138.1|ad-01|Active Directory|
|ad-02|Windows Server 2019|192.168.138.2|ad-02|Active Directory|
|ansible-01|CentOS 8|192.168.138.5|ansible-01|ansible砲|

## ■ ホストの追加
1. inventoryに追加
2. (オプション)新機能ならグループを追加し、プレイブックを作成後、site.ymlに追記
3. グループ変数を定義
4. ホスト変数を定義
5. ワークフローを定義

## ■ 運用上の注意
- secret`SSH_PASS_SALT`の定期変更
- 方針: 極力サードパーティ製のモジュールやロールを使わない

## ■ MEMO
- actionsの並行実行
  - ansible-01から各ホストにSSHでansibleを実行すると直列になる
    - 遅い
    - セットアップが楽
  - self-hosted runnerで各ホストがansibleを実行すると並列になる
    - 速い
    - セットアップが苦

- グループの命名規則
  - できれば`<セグメント>_<機能>_<役割(あれば)>`とすべきかも(家の環境は大きくないので機能のみとしている)

- OSごとにブランチを切って管理するのもいいかも
- もともといるzabbixをansible傘下にいれる(監視項目の設定まで管理するつもりはない)
- yumでインストールしたansibleのnmcliモジュールが貧弱でmethodとかいじれない
  - もう少し扱いやすくなったら、グループ変数に`connections`という変数を定義したい
- linux_commonのhandlersがごちゃごちゃしているのでディストリとバージョンで分ける
- 本来なら、設定ファイルの文法チェックを通してからサービスの再起動をするべき(だけどやってない)
- pacemakerのモジュールがほしい...
  - GUIツールに逃げる 
- drbdのモジュールもほしい...
- yumモジュールでプロキシ指定はできない。
  - また、プロキシの環境変数(http_proxyなど)も(少なくとも`/etc/profile.d`配下で宣言するものは)見てない
  - yum.conf(dnf.conf)にproxyパラメータを追記することでどうにかできる。
- role アプリごとにサーバとクライアントでわける
