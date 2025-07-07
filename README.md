# network-practice

このリポジトリはDocker Composeを用いてDMZ(非武装地帯)と内部ネットワークを分離した仮想ネットワークを構築する"学習用"プロジェクトです。

---

※※注意事項※※  
- この環境はインターネットとは接続されていません。  
- セキュリティ対策は未実施です。  
- 実際の環境でこの構成をそのまま使用することは、重大なセキュリティリスクを伴います。  
- この環境は学習用です。※本プロジェクトでは現在、学習目的のため `docker run` による手動構築を行っています。  
　チーム開発や環境の再現性を重視する場合は、将来的に `Docker Compose` による管理への移行を推奨します。
  



### 構成の目的
- 座学ではできない「体験」をするために勉強の一環として構築しました。
- 公開する情報と守る情報を管理しながら通信する仕組みが、セキュリティ面で有効であると学んだためこのような構成を構築しました。
　

## Docker コンテナ一覧

```bash
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                      PORTS                                                    NAMES
fb431995603c   mysql:latest   "docker-entrypoint.s…"   8 seconds ago   Up 8 seconds                33060/tcp, 0.0.0.0:3307->3306/tcp, [::]:3307->3306/tcp   dmz-db-c
62d0ff70348f   nginx          "/docker-entrypoint.…"   3 days ago      Up 29 minutes               0.0.0.0:8081->80/tcp, [::]:8081->80/tcp                  my_local_server
0745b0683a77   mysql:latest   "docker-entrypoint.s…"   3 weeks ago     Up 29 minutes               3306/tcp, 33060/tcp                                      mydb
2e23393015d1   alpine         "/bin/sh"                3 weeks ago     Up 29 minutes                                                                        client_1
a6a29141d5e7   nginx          "/docker-entrypoint.…"   3 weeks ago     Up 29 minutes               80/tcp                                                   webserv
73045fce47d0   ubuntu         "/bin/bash"              3 weeks ago     Up 29 minutes                                                                        rensyuu
```

## 構成について

この学習用プロジェクトは、内部ネットワークとDMZを意識して作ったものです。

### コンテナの区分
| コンテナ名         | 用途                     | 所属ネットワーク | 備考                              |
|--------------------|--------------------------|------------------|-----------------------------------|
| dmz-db-c           | データベースコンテナ       | DMZ用データベース（dmz-db）   | MySQL、rootパスワード設定あり                  |
| rensyuu            | 観測用コンテナ           | DMZ（prac-net）   | Ubuntuベース                      |
| webserv            | 外部公開用Webサーバ      | DMZ（prac-net）   | nginx使用                         |
| client_1           | クライアント端末         | 内部ネットワーク（client-net） | alpine使用                        |
| mydb               | 内部向けデータベースコンテナ     | 内部ネットワーク（client-net） | MySQL、rootパスワード設定あり     |
| my_local_server    | 内部向けサーバ           | 内部ネットワーク（client-net） | nginx使用、ポート8081で公開       |




## ネットワーク一覧
```bash
$  docker network ls
NETWORK ID     NAME                           DRIVER    SCOPE
b6efb3f30ff0   bridge                         bridge    local
a908670cf30b   client-net                     bridge    local
e0e6aa3e4816   dmz-db                         bridge    local
15f6afe0212b   docker_network_project_mynet   bridge    local
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
| dmz-db          　　　|　dmz用データベース、webサーバ　　　　　　　　　　　　   | 使用中           |


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
  
  外に公開するサイトなどの管理

```bash

$docker network create prac-net

```
- 観測用コンテナの作成

  webサーバなどへのアクセス確認用に作成。

```bash

$docker run -dit --name rensyuu --network prac-net ubuntu:20.04

```

- webサーバの作成

  webサイトの表示などの制御用で作成。

```bash

$docker run -dit --name webserv --network prac-net nginx

```

- DMZ用データベース作成
  -ネットワーク作成
  ```bash
  $docker network create dmz-db

  ```

- データベースコンテナ作成

 ※password=　の後には、任意のパスワードを書く

 ```bash
  
 $ docker run -d \
   --name dmz-db-c \
   -e MYSQL_ROOT_PASSWORD= \      
   -e MYSQL_DATABASE=mydatabase \
   -e MYSQL_USER=user1 \
   -e MYSQL_PASSWORD=userpass \
   --network dmz-db \
   -p 3307:3306 \
  mysql:latest

  ```



- DMZ用に作成したwebサーバを同じネットワークに追加
   ```bash
   $ docker network connect dmz-db webserv
   ```
  

## 内部ネットワークの構築の流れ

- 内部ネットワークの作成

　外に公開しない情報を扱うネットワークの作成。
  
```bash
$docker network create client-net

```
- クライアントのコンテナの作成
  
  内部ネットワークに接続されたクライアント端末を模したコンテナの作成。
  
  疎通確認やアプリケーションの動作検証などに使用。
 

```bash
$docker run -dit --name client_1 --network client-net alpine

```

- データベースのコンテナ作成

  外部に公開しない情報を入れるためのコンテナ作成。
  
```bash
$docker run -dit --name mydb --network client-net -e MYSQL_ROOT_PASSWORD=oajsmi mysql:latest

```

- サーバの作成

  外に公開しない情報への通信をするためのコンテナとして作成。

```bash
$docker run -dit --name my_local_server --network client-net -p 8081:80 nginx

```

## 通信の確認

- ネットワークが正常に構成されているかどうかを確認するため、DMZ用ネットワークおよび内部ネットワークにおける通信テストを実施した。

---

- DMZ用ネットワークの動作確認
   - 以下のコマンドで、rensyuu(テスト用コンテナ)に入る。
   
  
```bash
$ docker exec -it rensyuu bash

```
- そして、そのコンテナの中でrensyuu(webサーバ)にhttpリクエストを試みる。
```bash
$ docker exec -it rensyuu bash
root@73045fce47d0:/# curl http://webserv

```

結果は以下のとおりである。

```bash
$ docker exec -it rensyuu bash
root@73045fce47d0:/# curl http://webserv
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```
正しく表示された。


- 内部ネットワークの動作確認
  
  - 以下のコマンドを打ち、クライアントのコンテナに入る
```bash
$ docker exec -it client_1 sh

```

クライアントコンテナの中で、以下のコマンドを入力(mydbにpingを送る)

※このコマンドは、通信を永遠と行うため、ctrl+Cで止める必要がある。

```bash
$ ping mydb


```

その結果を以下に示す。


```bash
$ docker exec -it client_1 sh

/ # ping mydb
PING mydb (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.242 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.093 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.118 ms
64 bytes from 172.19.0.3: seq=3 ttl=64 time=0.128 ms
64 bytes from 172.19.0.3: seq=4 ttl=64 time=0.087 ms
64 bytes from 172.19.0.3: seq=5 ttl=64 time=0.089 ms
64 bytes from 172.19.0.3: seq=6 ttl=64 time=0.085 ms
64 bytes from 172.19.0.3: seq=7 ttl=64 time=0.084 ms
64 bytes from 172.19.0.3: seq=8 ttl=64 time=0.089 ms
64 bytes from 172.19.0.3: seq=9 ttl=64 time=0.089 ms
64 bytes from 172.19.0.3: seq=10 ttl=64 time=0.116 ms
64 bytes from 172.19.0.3: seq=11 ttl=64 time=0.194 ms
64 bytes from 172.19.0.3: seq=12 ttl=64 time=0.107 ms
64 bytes from 172.19.0.3: seq=13 ttl=64 time=0.091 ms
64 bytes from 172.19.0.3: seq=14 ttl=64 time=0.098 ms
^C
--- mydb ping statistics ---
15 packets transmitted, 15 packets received, 0% packet loss
round-trip min/avg/max = 0.084/0.114/0.242 ms

```

15回通信を試みた。また、15回すべてで、通信は成功している。


- DMZ用データベースの通信の確認

  これについては未実施である。

## ポートスキャン

それぞれのコンテナについてのポートを調べる。

---

- dmz-db-c

```bash
$ nmap -p- 172.20.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:23 JST
Nmap scan report for 172.20.0.2
Host is up (0.000074s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT      STATE SERVICE
3306/tcp  open  mysql
33060/tcp open  mysqlx

Nmap done: 1 IP address (1 host up) scanned in 2.30 seconds

```


- my_local_server
```bash
$ nmap -p- 172.19.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:24 JST
Nmap scan report for 172.19.0.2
Host is up (0.000069s latency).
Not shown: 65534 closed tcp ports (conn-refused)
＼PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.86 seconds
```

- mydb

```bash
$ nmap -p- 172.19.0.3
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:25 JST
Nmap scan report for 172.19.0.3
Host is up (0.000072s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT      STATE SERVICE
3306/tcp  open  mysql
33060/tcp open  mysqlx

Nmap done: 1 IP address (1 host up) scanned in 2.09 seconds

```

- client_1

```bash

$ nmap -p- 172.19.0.4
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:26 JST
Nmap scan report for 172.19.0.4
Host is up (0.000077s latency).
All 65535 scanned ports on 172.19.0.4 are in ignored states.
Not shown: 65535 closed tcp ports (conn-refused)

Nmap done: 1 IP address (1 host up) scanned in 2.08 seconds

```

- webserv (dmz-dbネットワーク)

```bash
$ nmap -p- 172.20.0.3
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:26 JST
Nmap scan report for 172.20.0.3
Host is up (0.000094s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.74 seconds
```

- webserv (prac-netネットワーク)


```bash

$ nmap -p- 172.18.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:27 JST
Nmap scan report for 172.18.0.2
Host is up (0.000074s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.83 seconds

```

 - rensyuu


```bash
$ nmap -p- 172.18.0.3
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-07-04 16:28 JST
Nmap scan report for 172.18.0.3
Host is up (0.000067s latency).
All 65535 scanned ports on 172.18.0.3 are in ignored states.
Not shown: 65535 closed tcp ports (conn-refused)

Nmap done: 1 IP address (1 host up) scanned in 2.24 seconds
```

# Nmapスキャン結果まとめ

## DMZネットワーク（dmz-db）

| コンテナ名   | IPアドレス   | 開いているポート            | 備考             |
|--------------|--------------|-----------------------------|------------------|
| dmz-db-c     | 172.20.0.2   | 3306/tcp, 33060/tcp         | MySQL稼働中      |
| webserv      | 172.20.0.3   | 80/tcp                      | HTTP稼働中       |

## 内部ネットワーク（prac-net）

| コンテナ名     | IPアドレス   | 開いているポート            | 備考               |
|----------------|--------------|-----------------------------|--------------------|
| mydb           | 172.19.0.3   | 3306/tcp, 33060/tcp         | MySQL稼働中        |
| my_local_server| 172.19.0.2   | 80/tcp                      | Webサーバ稼働中    |
| client_1       | 172.19.0.4   | なし（全ポート閉じている） | 接続検証未実施     |

## その他ネットワーク（prac-netなど）

| コンテナ名 | IPアドレス   | ポート        | 備考         |
|------------|--------------|---------------|--------------|
| webserv    | 172.18.0.2   | 80/tcp        | 複数NW所属    |
| rensyuu    | 172.18.0.3   | なし          | 実験用        |

---

## 通信確認状況

- `dmz-db-c` → **Nmapでポート開放は確認済み**
- **MySQL接続（例：mysql -h ...）は未実施**
- `client_1` → ping確認済み（通信可能）
- `webserv` → DMZ/内部ネットワークの両方でWebアクセス可能


## 所感

知識がほぼゼロの状態から、「こういうネットワークを作りたい」という構想だけをAIに伝えるところからスタートしました。  
返ってきた提案を参考にしながら、自らコマンドを打ち、出力やエラーと向き合い、必要な知識を都度学びながら環境を構築していきました。

完成した構成については、コンテナごとの役割や通信経路、ネットワークの分離意図を自分の言葉で説明できるようになり、表や記録も含めてまとめました。  
単なる動作確認にとどまらず、nmapによるポートスキャンなどから「どのコンテナが、どのポートでサービスを提供しているか」まで把握できるようになりました。

現在は、得られた知見をもとにファイアウォールの設置や通信制御の検討に取り組んでおり、ネットワークの基本的なセキュリティ設計にも一歩踏み込んで学習を進めています。


