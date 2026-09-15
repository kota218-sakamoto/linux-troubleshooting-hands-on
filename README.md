# Linux Troubleshooting Hands-on

Linux環境で代表的な障害を意図的に再現し、事象確認・切り分け・原因特定・復旧・正常性確認まで実施したトラブルシューティングハンズオンです。

## 目的

Linuxサーバーで発生する代表的な障害に対して、コマンドの実行だけではなく、事象から原因を段階的に切り分ける流れを身につけることを目的としています。

## 検証環境

| ホスト | IPアドレス | 主な役割 |
|---|---|---|
| WEB01 | 192.168.100.10 | Ubuntu / Apache / PHP |
| DB01 | 192.168.100.20 | Ubuntu / PostgreSQL 18 |

WEB01からDB01上のPostgreSQLへ接続し、Webページからデータベースの内容を表示する構成を使用しています。

## 実施した障害対応

| No. | 障害 | 主な確認コマンド |
|---|---|---|
| 1 | Apache設定構文エラーによる起動障害 | `systemctl`, `apache2ctl configtest` |
| 2 | PostgreSQL接続許可設定ミス | `nc`, `psql` |
| 3 | ファイル権限ミスによる403 Forbidden | `curl`, `ls -l` |
| 4 | ディスク使用率逼迫 | `df`, `du`, `findmnt` |
| 5 | PostgreSQLサービス停止によるポート待受障害 | `nc`, `pg_lsclusters`, `ss` |
## 障害対応ドキュメント

### 1. Apache起動障害

Apache設定ファイルに不正なディレクティブを追加し、サービス起動失敗を再現しました。

`systemctl status` と `apache2ctl configtest` を使用して原因箇所を特定し、設定修正後にサービスおよびWebページの復旧を確認しました。

[詳細](docs/apache-startup-failure.md)

### 2. PostgreSQL接続許可設定ミス

`pg_hba.conf` のWEB01向け接続許可を無効化し、WebアプリケーションからDBへ接続できない状態を再現しました。

TCP/5432への到達性を確認した後、`psql` のエラーからPostgreSQL側の接続許可設定を原因として特定しました。

[詳細](docs/postgresql-access-failure.md)

### 3. ファイル権限ミス

Webコンテンツの権限を変更し、Apacheから読み取れない状態にすることで403 Forbiddenを再現しました。

HTTPレスポンスとファイル権限を確認し、`chmod` で適切な権限へ戻した後に200 OKを確認しました。

[詳細](docs/file-permission-error.md)

### 4. ディスク使用率逼迫

検証用の小容量ファイルシステムを作成し、大容量ファイルによってディスク使用率96%の状態を再現しました。

`df` で逼迫しているファイルシステムを確認し、`du` で容量を消費しているファイルを特定して復旧しました。

[詳細](docs/disk-usage-incident.md)

### 5. PostgreSQLサービス停止

PostgreSQLクラスタを停止し、TCP/5432の待受がなくなる障害を再現しました。

`nc` の Connection refused を起点に、`pg_lsclusters` と `ss` を使用してサービス停止とポート待受消失を確認しました。

[詳細](docs/postgresql-service-down.md)

## トラブルシューティングの基本フロー

    障害発生
        ↓
    事象確認
        ↓
    影響範囲確認
        ↓
    切り分け
        ↓
    ログ・設定・サービス状態確認
        ↓
    原因特定
        ↓
    復旧
        ↓
    正常性確認

## 使用した主なコマンド

    systemctl
    apache2ctl
    curl
    nc
    psql
    pg_lsclusters
    ss
    ls
    chmod
    df
    du
    findmnt

## 学んだこと

障害対応では、最初から設定ファイルを変更するのではなく、まず事象と影響範囲を確認し、ネットワーク・サービス・設定・権限・リソースなどの観点から段階的に切り分けることが重要だと確認しました。

また、同じDB接続障害でも、接続許可設定の問題ではTCP/5432への接続が成功する一方、サービス停止時には Connection refused となるなど、エラー内容やポート状態から原因の範囲を絞り込めることを検証しました。

## 証跡ログ

各障害について、障害発生時と復旧後の実行結果を保存しています。

| 障害 | 障害時 | 復旧後 |
|---|---|---|
| Apache起動障害 | [failure.txt](evidence/apache/failure.txt) | [recovery.txt](evidence/apache/recovery.txt) |
| PostgreSQL接続許可設定ミス | [failure.txt](evidence/postgresql-access/failure.txt) | [recovery.txt](evidence/postgresql-access/recovery.txt) |
| ファイル権限ミス | [failure.txt](evidence/permission/failure.txt) | [recovery.txt](evidence/permission/recovery.txt) |
| ディスク使用率逼迫 | [failure.txt](evidence/disk/failure.txt) | [recovery.txt](evidence/disk/recovery.txt) |
| PostgreSQLサービス停止 | [failure.txt](evidence/postgresql-service/failure.txt) | [recovery.txt](evidence/postgresql-service/recovery.txt) |
