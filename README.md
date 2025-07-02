# network-practice

このリポジトリはDocker Composeを用いてDMZ(非武装地帯)と内部ネットワークを分離した仮想ネットワークを構築する"学習用"プロジェクトです。

---

※※注意事項※※  
- この環境はインターネットとは接続されていません。  
- セキュリティ対策は未実施です。  
- 実際の環境でこの構成をそのまま使用することは、重大なセキュリティリスクを伴います。  
- この環境は学習用です。

```

## Docker コンテナ一覧

```bash
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED       STATUS              PORTS                                                  NAMES
62d0ff70348f   nginx           "/docker-entrypoint.…"   7 hours ago   Up 2 minutes        0.0.0.0:8081->80/tcp, [::]:8081->80/tcp                my_local_server
0745b0683a77   mysql:latest    "docker-entrypoint.s…"   2 weeks ago   Up About a minute   3306/tcp, 33060/tcp                                    mydb
2e23393015d1   alpine          "/bin/sh"                2 weeks ago   Up About a minute                                                          client_1
a6a29141d5e7   nginx           "/docker-entrypoint.…"   3 weeks ago   Up About a minute   80/tcp                                                 webserv
73045fce47d0   ubuntu          "/bin/bash"              3 weeks ago   Up About a minute                                                          rensyuu
```

## 構成について

この学習用プロジェクトは、内部ネットワークとDMZを意識して作ったものです。

### コンテナの区分
| コンテナ名         | 用途                     | 所属ネットワーク | 備考                              |
|--------------------|--------------------------|------------------|-----------------------------------|
| rensyuu            | 観測用コンテナ           | DMZ（prac-net）   | Ubuntuベース                      |
| webserv            | 外部公開用Webサーバ      | DMZ（prac-net）   | nginx使用                         |
| client_1           | クライアント端末         | 内部ネットワーク（client-net） | alpine使用                        |
| mydb               | データベースコンテナ     | 内部ネットワーク（client-net） | MySQL、rootパスワード設定あり     |
| my_local_server    | 内部向けサーバ           | 内部ネットワーク（client-net） | nginx使用、ポート8081で公開       |




## ネットワーク一覧
```bash
$ docker network ls
NETWORK ID     NAME                           DRIVER    SCOPE
152dad7fe181   bridge                         bridge    local
a908670cf30b   client-net                     bridge    local
ab501f221595   host                           host      local
6d5131446d23   none                           null      local
09f9b00bb76a   prac-net                       bridge    local
f35992645f42   system32_mynet                 bridge    local


```
## 各ネットワークについて

bridge、host、noneは,いずれもDocker標準のデフォルトネットワークである。

| ネットワーク名       | 用途・役割                                         | 使用状況         |
|----------------------|----------------------------------------------------|------------------|
| bridge               | Docker標準のデフォルトネットワーク                 | 自動作成 / 使用中 |
| host                 | ホストネットワーク（コンテナとホストを統一）       | 自動作成 / 使用中 |
| none                 | ネットワークなし                                   | 自動作成 / 使用中 |
| prac-net             | DMZ（非武装地帯）用。Webサーバ・観測用コンテナ用   | 使用中           |
| client-net           | 内部ネットワーク。クライアント・DB・内部サーバ用   | 使用中           |
| system32_mynet       | 過去の学習で作成したネットワーク                   | **未使用**        |


---

## 構築をする際の環境設定
- wslのインストール
  - 初回セットアップ時、パスワードを設定する必要があります。このパスワードは、Linux環境で'sudo'コマンドを使う際に必要です。
  - このパスワードはお使いのデバイスにログインする際のパスワードとは別物です。
```bash

$wsl --install

```

  - wsl --install` コマンド実行後は、**PCの再起動が必要**です。
    
- wslインスタンスに入る
  - 再起動しないとwslの機能が正しく動作しない場合があります。
  - 再起動後に、再度操作を試みてください
    
```bash

$wsl --distribution Ubuntu

```
以下の構築の流れは、wslインスタンスに入った後に行う操作である。

## 構築の流れ

- DMZ用ネットワークの作成

```bash

$docker network create prac-net

```
- 観測用コンテナの作成

```bash

$docker run -dit --name rensyuu --network prac-net ubuntu:20.04

```

- webサーバの作成

```bash

$docker run -dit --name webserv --network prac-net nginx

```

## 内部ネットワークの構築の流れ

- 内部ネットワークの作成
```bash
$docker network create client-net

```
- クライアントのコンテナの作成

```bash
$docker run -dit --name client_1 --network client-net alpine

```

- データベースのコンテナ作成

```bash
$docker run -dit --name mydb --network client-net -e MYSQL_ROOT_PASSWORD=oajsmi mysql:latest

```

- サーバの作成

```bash
$docker run -dit --name my_local_server --network client-net -p 8081:80 nginx

```

  
