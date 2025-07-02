# network-practice

このリポジトリはDocker Composeを用いて仮想ネットワークを構築する"学習用"プロジェクトです。

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

- **DMZ**（非武装地帯）
  -rensyuu　(観測用コンテナ)
  - webserv (Webサーバ)

- **内部ネットワーク**
  - client_1 (クライアント)
  - mydb (データベース)
  - my_local_server (サーバ)

### 役割・説明

- **webserv**
  
  外部からのアクセスを受けるWebサーバ。
  
- **client_1**
  
  内部ネットワークのクライアント端末。
   
- **mydb**
  
  データベースコンテナ。
  
- **my_local_server**
  
  内部でのサーバ機能を持つコンテナ。  

```
##ネットワーク一覧
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
###各ネットワークについて

--Dockerをインストールすると入ってるネットワーク--

bridge

host

none                          

--DMZを構築する用のネットワーク--

　　prac-net     

--内部ネットワーク(クライアント、データベース、サーバのコンテナを構築する用)--
　　
  
  　client-net                   
                     
--現在使用してないネットワーク--

以下のネットワークは過去の学習で作成されたもので、現在は使われてません

　　system32_mynet               

