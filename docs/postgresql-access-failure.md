# PostgreSQL接続許可設定ミス

## 概要

PostgreSQLの接続許可設定である `pg_hba.conf` の設定を変更し、WEB01からDB01へ接続できない障害を再現した。

## 障害内容

DB01の `/etc/postgresql/18/main/pg_hba.conf` で、WEB01からの接続を許可している以下の設定をコメントアウトした。

    # host    webappdb    webuser    192.168.100.10/32    scram-sha-256

設定変更後、PostgreSQLへ設定を反映した。

    sudo systemctl reload postgresql

その結果、WEB01のDB連携ページからPostgreSQLへ接続できなくなった。

    curl http://localhost/dbtest.php

結果：

    Database connection failed.

## 切り分け

まずWEB01からDB01のTCP/5432へ接続できるか確認した。

    nc -zv 192.168.100.20 5432

結果：

    Connection to 192.168.100.20 5432 port [tcp/postgresql] succeeded!

TCP/5432への接続は成功しているため、ネットワーク疎通やPostgreSQLのポート待受には問題がないと判断した。

次に、WEB01からPostgreSQLへ直接接続した。

    psql -h 192.168.100.20 -U webuser -d webappdb

結果：

    FATAL: no pg_hba.conf entry for host "192.168.100.10", user "webuser", database "webappdb"

このエラーから、`pg_hba.conf` にWEB01からの接続を許可する設定が存在しないことが原因と判断した。

## 復旧

DB01の `/etc/postgresql/18/main/pg_hba.conf` に以下の設定を戻した。

    host    webappdb    webuser    192.168.100.10/32    scram-sha-256

設定変更後、PostgreSQLへ設定を反映した。

    sudo systemctl reload postgresql

## 復旧確認

WEB01からPostgreSQLへ直接接続した。

    psql -h 192.168.100.20 -U webuser -d webappdb

接続に成功し、`webappdb=>` プロンプトが表示された。

続いてDB連携ページを確認した。

    curl http://localhost/dbtest.php

Employee Listが正常に表示され、WEB01からDB01への接続が復旧したことを確認した。

## 原因

`pg_hba.conf` のWEB01用接続許可設定が無効になっていたため、PostgreSQLがWEB01からの接続を拒否していた。

## 対応のポイント

DB接続障害では、まず `nc` などを使用してTCPポートへの到達性を確認する。

TCP/5432へ接続できる場合は、ネットワーク障害ではなく、PostgreSQLの認証設定や接続許可設定などを確認する。

`psql` で直接接続することで、アプリケーションを介さずPostgreSQLが返すエラーを確認できる。
