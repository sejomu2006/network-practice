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
d202ab96982e   nginx:latest    "/docker-entrypoint.…"   6 days ago    Up 2 minutes        0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                docker_network_project_web_1
8ec36a408637   mysql:5.7       "docker-entrypoint.s…"   6 days ago    Up 2 minutes        0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   docker_network_project_db_1
36080defcebe   alpine:latest   "sh -c 'while true; …"   6 days ago    Up About a minute                                                          docker_network_project_client_1
a4f992aa6c47   alpine:latest   "sh -c 'while true; …"   2 weeks ago   Up About a minute                                                          system32_client_1
d9d9d40218bf   alpine          "sh"                     2 weeks ago   Up About a minute                                                          client
dae2c5e26a6b   mysql:5.7       "docker-entrypoint.s…"   2 weeks ago   Up About a minute   3306/tcp, 33060/tcp                                    db
0745b0683a77   mysql:latest    "docker-entrypoint.s…"   2 weeks ago   Up About a minute   3306/tcp, 33060/tcp                                    mydb
2e23393015d1   alpine          "/bin/sh"                2 weeks ago   Up About a minute                                                          client_1
a6a29141d5e7   nginx           "/docker-entrypoint.…"   3 weeks ago   Up About a minute   80/tcp                                                 webserv
73045fce47d0   ubuntu          "/bin/bash"              3 weeks ago   Up About a minute                                                          rensyuu
```

## 構成について

この学習用プロジェクトは、内部ネットワークとDMZを意識して作ったものです。

### コンテナの区分

- **DMZ**（非武装地帯）
  - webserv (Webサーバ)

- **内部ネットワーク**
  - client_1 (クライアント)
  - mydb (データベース)
  - locl_serv (サーバ)

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
15f6afe0212b   docker_network_project_mynet   bridge    local
ab501f221595   host                           host      local
6d5131446d23   none                           null      local
09f9b00bb76a   prac-net                       bridge    local
f35992645f42   system32_mynet                 bridge    local
d3406781de7b   test-net                       bridge    local

```
###各ネットワークについて

--Dockerをインストールすると入ってるネットワーク--

152dad7fe181   bridge                         bridge    local

ab501f221595   host                           host      local

6d5131446d23   none                           null      local

--DMZを構築する用のネットワーク--

　　prac-net     

--内部ネットワーク(クライアント、データベース、サーバのコンテナを構築する用)--
　　
  
  client-net                   

--内部ネットワークの通信確認用ネットワーク--

　　test-net                      

--現在使用してないネットワーク--

以下のネットワークは過去の学習で作成されたもので、現在は使われてません

　　system32_mynet               

