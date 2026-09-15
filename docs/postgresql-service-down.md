# PostgreSQLサービス停止によるポート待受障害

## 概要

PostgreSQLクラスタを停止し、TCP/5432の待受がなくなることでWEB01からDB01へ接続できない障害を再現した。

## 障害内容

DB01でPostgreSQL 18のクラスタを停止した。

    sudo systemctl stop postgresql@18-main

停止後、WEB01からDB01のTCP/5432へ接続を試みた。

    nc -zv 192.168.100.20 5432

結果：

    nc: connect to 192.168.100.20 port 5432 (tcp) failed: Connection refused

TCP/5432への接続が拒否されることを確認した。

## 切り分け

DB01でPostgreSQLクラスタの状態を確認した。

    pg_lsclusters

結果：

    Ver Cluster Port Status Owner    Data directory               Log file
    18  main    5432 down   postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log

PostgreSQLクラスタが `down` になっていることを確認した。

続いて、TCP/5432の待受状態を確認した。

    sudo ss -lntp | grep 5432

結果：

    出力なし

TCP/5432をLISTENしているプロセスが存在しないことから、PostgreSQLクラスタ停止が接続障害の原因と判断した。

## 復旧

PostgreSQLクラスタを起動した。

    sudo systemctl start postgresql@18-main

クラスタ状態を確認した。

    pg_lsclusters

TCP/5432の待受状態も確認した。

    sudo ss -lntp | grep 5432

PostgreSQLクラスタが起動し、TCP/5432がLISTEN状態になっていることを確認した。

## 復旧確認

WEB01からPostgreSQLへ直接接続した。

    psql -h 192.168.100.20 -U webuser -d webappdb

接続に成功し、以下のプロンプトが表示された。

    webappdb=>

続いてDB連携ページを確認した。

    curl http://localhost/dbtest.php

Employee Listが正常に表示され、WEB01からDB01への接続が復旧したことを確認した。

## 原因

PostgreSQL 18の `main` クラスタが停止していたため、TCP/5432で待受するプロセスが存在せず、WEB01からの接続が拒否されていた。

## 対応のポイント

DB接続障害では、まずTCPポートへの到達性を確認する。

`Connection refused` の場合は、対象ホストには到達できているものの、そのポートで待受しているサービスが存在しない可能性がある。

そのため、`pg_lsclusters` や `systemctl` でPostgreSQLの状態を確認し、`ss` で実際のポート待受状態を確認する。

`pg_hba.conf` の設定ミスのようにTCP接続自体は成功する障害と、サービス停止によってポート待受そのものが存在しない障害を切り分けることが重要である。
