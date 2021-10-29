# グループ変数
## 概要
|変数名|yml|OS|目的|
|:---|:---|:---|:---|
|resolv|set_dns.yml|CentOS 8|resolv.confに記載する情報を記述する|
|ntpservers|set_ntp.yml|WindowsServer 2019|参照するNTPサーバを記述する|
|synctype|set_ntp.yml|WindowsServer 2019|時刻同期の挙動を記述する|

## 構造
変数は以下の構造で定義する。
- 機能名
  - 変数名
    - 要素1
    - 要素2
