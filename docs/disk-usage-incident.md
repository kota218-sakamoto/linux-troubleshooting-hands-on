# ディスク使用率逼迫

## 概要

検証用の小容量ファイルシステムを作成し、大容量ファイルを配置することでディスク使用率の逼迫を再現した。

## 検証環境

本番環境のルートファイルシステムへ影響を与えないよう、200MBのイメージファイルを作成してループデバイスとしてマウントした。

    sudo mkdir -p /mnt/disk-test
    sudo truncate -s 200M /tmp/disk-test.img
    sudo mkfs.ext4 -F /tmp/disk-test.img
    sudo mount -o loop /tmp/disk-test.img /mnt/disk-test

マウント状態を確認した。

    df -h /mnt/disk-test
    findmnt /mnt/disk-test

確認結果：

    /dev/loop3  172M  152K  158M  1%  /mnt/disk-test

## 障害内容

検証用ファイルシステム内に150MBのファイルを作成した。

    sudo fallocate -l 150M /mnt/disk-test/large.log

## 事象確認

ディスク使用率を確認した。

    df -h /mnt/disk-test

結果：

    Filesystem  Size  Used  Avail  Use%  Mounted on
    /dev/loop3  172M  151M  7.2M   96%  /mnt/disk-test

ディスク使用率が96%まで上昇していることを確認した。

## 切り分け

容量を消費しているファイルを調査した。

    sudo du -ah /mnt/disk-test | sort -rh | head

結果：

    151M    /mnt/disk-test
    150M    /mnt/disk-test/large.log
    16K     /mnt/disk-test/lost+found

`large.log` が150MBを使用しており、ディスク逼迫の主な原因であることを特定した。

## 復旧

検証用の大容量ファイルを削除した。

    sudo rm /mnt/disk-test/large.log

## 復旧確認

ディスク使用率を再確認した。

    df -h /mnt/disk-test

結果：

    Filesystem  Size  Used  Avail  Use%  Mounted on
    /dev/loop3  172M  152K  158M    1%  /mnt/disk-test

容量を使用しているファイルも再確認した。

    sudo du -ah /mnt/disk-test | sort -rh | head

結果：

    20K    /mnt/disk-test
    16K    /mnt/disk-test/lost+found

ディスク使用率が96%から1%まで低下し、復旧したことを確認した。

## 後片付け

検証終了後、ループデバイスとイメージファイルを削除した。

    sudo umount /mnt/disk-test
    sudo rm /tmp/disk-test.img
    sudo rmdir /mnt/disk-test

以下のコマンドで後片付けが完了していることを確認した。

    findmnt /mnt/disk-test
    ls -l /tmp/disk-test.img

## 原因

`large.log` がファイルシステム容量の大部分を使用していたため、ディスク使用率が96%まで上昇していた。

## 対応のポイント

ディスク容量逼迫時は、まず `df -h` で逼迫しているファイルシステムを特定する。

その後、`du` などを使用してディレクトリやファイル単位で容量を確認し、容量を消費している対象を特定する。

原因を確認せずにファイルを削除するのではなく、影響やファイルの用途を確認したうえで対応することが重要である。
