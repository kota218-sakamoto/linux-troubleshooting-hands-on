# Apache起動障害

## 概要

Apacheの設定ファイルに不正なディレクティブを追加し、サービスが起動できない障害を再現した。

## 障害内容

`/etc/apache2/apache2.conf` に以下の不正な設定を追加した。

    InvalidDirectiveTest

Apacheを再起動すると起動に失敗した。

    sudo systemctl restart apache2

## 事象確認

以下のコマンドでApacheの状態を確認した。

    systemctl status apache2

確認結果：

    Active: failed
    AH00526: Syntax error on line 226 of /etc/apache2/apache2.conf
    Invalid command 'InvalidDirectiveTest'

## 切り分け

Apacheの設定ファイルに構文エラーがないか確認した。

    sudo apache2ctl configtest

結果：

    AH00526: Syntax error on line 226 of /etc/apache2/apache2.conf:
    Invalid command 'InvalidDirectiveTest'

これにより、Apache設定ファイルの226行目にある不正なディレクティブが原因と判断した。

## 復旧

`/etc/apache2/apache2.conf` から以下の行を削除した。

    InvalidDirectiveTest

設定を再確認した。

    sudo apache2ctl configtest

結果：

    Syntax OK

Apacheを再起動した。

    sudo systemctl restart apache2

## 復旧確認

以下のコマンドでApacheの状態を確認した。

    systemctl status apache2

結果：

    Active: active (running)

Webページへのアクセスも確認した。

    curl http://localhost

結果：

    <h1>Welcome to web01</h1>

DB連携ページも正常に表示できることを確認した。

    curl http://localhost/dbtest.php

## 原因

Apache設定ファイルに存在しないディレクティブが記述されていたため、設定ファイルの読み込みに失敗し、Apacheが起動できなかった。

## 対応のポイント

サービス起動失敗時は、まず `systemctl status` で状態を確認し、Apacheの場合は `apache2ctl configtest` を使用して設定ファイルの構文を確認する。
