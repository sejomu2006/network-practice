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
$ docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED       STATUS                       PORTS                                                  NAMES
d202ab96982e   nginx:latest    "/docker-entrypoint.…"   5 days ago    Exited (255) 9 seconds ago   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                docker_network_project_web_1
8ec36a408637   mysql:5.7       "docker-entrypoint.s…"   5 days ago    Exited (255) 9 seconds ago   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   docker_network_project_db_1
36080defcebe   alpine:latest   "sh -c 'while true; …"   5 days ago    Exited (255) 9 seconds ago                                                          docker_network_project_client_1
a4f992aa6c47   alpine:latest   "sh -c 'while true; …"   2 weeks ago   Exited (255) 9 days ago                                                             system32_client_1
9f361f9b4ad7   nginx:latest    "/docker-entrypoint.…"   2 weeks ago   Created                                                                             9f361f9b4ad7_system32_web_1
d9d9d40218bf   alpine          "sh"                     2 weeks ago   Exited (255) 9 days ago                                                             client
dae2c5e26a6b   mysql:5.7       "docker-entrypoint.s…"   2 weeks ago   Exited (255) 9 days ago      3306/tcp, 33060/tcp                                    db
0e88740c8fa7   nginx           "/docker-entrypoint.…"   2 weeks ago   Exited (255) 9 days ago      0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                locl_serv
0745b0683a77   mysql:latest    "docker-entrypoint.s…"   2 weeks ago   Exited (255) 9 days ago      3306/tcp, 33060/tcp                                    mydb
2e23393015d1   alpine          "/bin/sh"                2 weeks ago   Exited (255) 9 days ago                                                             client_1
a6a29141d5e7   nginx           "/docker-entrypoint.…"   3 weeks ago   Exited (255) 9 days ago      80/tcp                                                 webserv
73045fce47d0   ubuntu          "/bin/bash"              3 weeks ago   Exited (255) 9 days ago                                                             rensyuu
5d247f373fa2   hello-world     "/hello"                 3 weeks ago   Exited (0) 2 weeks ago　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　  suspicious_lovelace

---

## 構成について

この学習用プロジェクトは、内部ネットワークとDMZを意識して作ったものです。

### ネットワークの区分

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
- **locl_serv**  
  内部のサーバ機能を持つコンテナ。  

### 注意点

- この環境はインターネット非接続です。  
- セキュリティ対策は未実施のため実運用には適しません。  
- 学習用の構成としてのみ使用してください。
