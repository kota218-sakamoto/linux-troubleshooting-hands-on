# ファイル権限ミスによる403 Forbidden

## 概要

Webコンテンツのファイル権限を変更し、Apacheからファイルを読み取れない状態を作ることで403 Forbiddenを再現した。

## 障害内容

検証用ファイルを作成した。

    echo "<h1>Permission Test</h1>" | sudo tee /var/www/html/permission-test.html

正常に表示できることを確認した。

    curl http://localhost/permission-test.html

結果：

    <h1>Permission Test</h1>

その後、検証用ファイルの権限をすべて削除した。

    sudo chmod 000 /var/www/html/permission-test.html

再度アクセスした。

    curl -i http://localhost/permission-test.html

結果：

    HTTP/1.1 403 Forbidden

## 切り分け

ApacheからHTTP応答自体は返っているため、Apacheサービス停止ではないと判断した。

対象ファイルの権限を確認した。

    ls -l /var/www/html/permission-test.html

結果：

    ---------- 1 root root 25 Sep 15 09:22 /var/www/html/permission-test.html

所有者、グループ、その他のユーザーすべてに権限が設定されていないことを確認した。

Apacheから対象ファイルを読み取れないことが403 Forbiddenの原因と判断した。

## 復旧

対象ファイルに読み取り可能な権限を設定した。

    sudo chmod 644 /var/www/html/permission-test.html

権限を確認した。

    ls -l /var/www/html/permission-test.html

結果：

    -rw-r--r-- 1 root root 25 Sep 15 09:22 /var/www/html/permission-test.html

## 復旧確認

Webページへ再度アクセスした。

    curl -i http://localhost/permission-test.html

結果：

    HTTP/1.1 200 OK

    <h1>Permission Test</h1>

HTTPステータスが200 OKとなり、正常にコンテンツを取得できることを確認した。

## 原因

Webコンテンツのファイル権限が `000` となっており、Apacheが対象ファイルを読み取れなかった。

## 対応のポイント

403 Forbiddenが発生した場合、Webサーバー自体が応答しているかを確認したうえで、対象ファイルやディレクトリの所有者・グループ・権限を確認する。

`ls -l` を使用することで、対象ファイルの権限状態を確認できる。
