# CentOS 8
## ■ 動作検証
|検証バージョン|動作検証状況|検証日|備考欄|
|:---|:---|:---|:---|
|8.3|OK|2021/11/02||
|8.5|OK|2021/11/02||
|8.6|OK|2021/11/02||

## ■ セットアップ
### ネットワーク
必要最低限の設定を入れる
```
$ nmcli connection modify <con-name> autoconnect yes
$ nmcli connection modify <con-name> ipv4.method manual ipv4.addresses <ip-addr> ipv4.gateway <gw>
$ nmcli connection up <con-name>
```
### ユーザ
```
$ useradd ansible
$ passwd ansible
```
```
$ vi /etc/pam.d/su
-  #
+  
```
```
$ usermod -aG wheel ansible
```

## ■ メモ
- SNMPは個別でロールを作りincludeさせる
