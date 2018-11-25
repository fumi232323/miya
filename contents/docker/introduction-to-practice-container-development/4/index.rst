.. article::
   :date: 2018-11-25
   :title: Docker/Kubernetes 実践コンテナ開発入門 --- 4. Swarm による実践的なアプリケーションの構築
   :category: docker
   :tags:
   :canonical_url: /docker/introduction-to-practice-container-development/3/
   :draft: true


=============================================
4. Swarm による実践的なアプリケーションの構築
=============================================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


4.1 Web アプリケーションの構成
==============================

つくるもの: TODO アプリ

.. list-table:: アーキテクチャの構成要素
  :widths: auto
  :header-rows: 1

  * - イメージ名
    - 用途
    - Service 名
    - Stack 名
  * - MySQL
    - データストア
    - mysql_master, mysql_slave
    - MySQL
  * - API
    - データストアを操作するAPIサーバー
    - app_api
    - Application
  * - Web
    - ビューを生成するアプリケーションサーバー
    - frontend_web
    - Frontend
  * - Nginx
    - プロキシサーバー
    - app_nginx, frontend_nginx
    - Application Frontend


1. 第３章で作成した Swarm 環境を利用する。

    .. code-block:: bash

      # 4つのノードで Swarm クラスタが実行されていることを確認しておく。
      $ docker container exec -it manager docker node ls
      ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
      u8crbubyz85jmnpou9zgnc1wf     272227e51007        Ready               Active                                  18.05.0-ce
      ww80hcmlzbga1cprtz0cmmuu2     7190447651de        Ready               Active                                  18.05.0-ce
      7f20ikf4s04lp9abasnvm8euz *   a7e4b99c1ee7        Ready               Active              Leader              18.05.0-ce
      j91gerxg4um0r05wukdfoqwo1     cfffba103c8c        Ready               Active                                  18.05.0-ce


2. 専用の overlay ネットワークを構築する。

    .. code-block:: bash

      $ docker container exec -it manager \
        docker network create --driver=overlay --attachable todoapp
        306ohjb606wx9atd7kxh0lf3a


4.1.3 TODO アプリケーション構築の全体像
---------------------------------------
1. データストアとなる Master/Slave 構成の MySQL Service の構築
2. MySQL とデータのやり取りをするための API 実装
3. ウェブアプリケーションと API 間にリバースプロキシとなる Nginx を通じてアクセスできるように設定
4. APIを利用してサーバーサイドレンダリングをする Web アプリケーションを実装
5. フロント側にリバースプロキシ (Nginx) を置く


4.2 MySQL Service の構築
========================

https://github.com/gihyodocker/tododb からクローン


4.2.1 データベースコンテナ構成
------------------------------


4.2.2 認証情報
--------------


4.2.5 MySQL の Dockerfile
-------------------------

ビルドと Swarm クラスタとしての利用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Docker イメージを ``ch04/tododb:latest`` という名前でビルドする。

.. code-block:: bash

  # tododb ディレクトリで実行する
  $ docker image build -t ch04/tododb:latest .
  Sending build context to Docker daemon  111.6kB
  Step 1/16 : FROM mysql:5.7
   ---> ae6b78bedf88
  Step 2/16 : RUN apt-get update
   ---> Running in 7ac5673dbad1
  Get:1 http://repo.mysql.com/apt/debian stretch InRelease [19.2 kB]
  Get:2 http://security-cdn.debian.org/debian-security stretch/updates InRelease [94.3 kB]
  Get:5 http://repo.mysql.com/apt/debian stretch/mysql-5.7 amd64 Packages [5675 B]
  Ign:3 http://cdn-fastly.deb.debian.org/debian stretch InRelease
  Get:4 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]
  Get:7 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 Packages [460 kB]
  Get:6 http://cdn-fastly.deb.debian.org/debian stretch Release [118 kB]
  Get:8 http://cdn-fastly.deb.debian.org/debian stretch-updates/main amd64 Packages [5152 B]
  Get:9 http://cdn-fastly.deb.debian.org/debian stretch Release.gpg [2434 B]
  Get:10 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 Packages [7089 kB]
  Fetched 7885 kB in 6s (1146 kB/s)
  Reading package lists...
  Removing intermediate container 7ac5673dbad1
   ---> d6b3838b06ce
  Step 3/16 : RUN apt-get install -y wget
   ---> Running in 2879665e7c48
  Reading package lists...
  Building dependency tree...
  Reading state information...
  The following additional packages will be installed:
    ca-certificates libidn2-0 libpsl5 libunistring0 publicsuffix
  The following NEW packages will be installed:
    ca-certificates libidn2-0 libpsl5 libunistring0 publicsuffix wget
  0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
  Need to get 1466 kB of archives.
  After this operation, 4977 kB of additional disk space will be used.
  Get:1 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libunistring0 amd64 0.9.6+really0.9.3-0.1 [279 kB]
  Get:2 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libidn2-0 amd64 0.16-1+deb9u1 [60.7 kB]
  Get:3 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libpsl5 amd64 0.17.0-3 [41.8 kB]
  Get:4 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 wget amd64 1.18-5+deb9u2 [799 kB]
  Get:5 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 ca-certificates all 20161130+nmu1+deb9u1 [182 kB]
  Get:6 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 publicsuffix all 20181003.1334-0+deb9u1 [104 kB]
  debconf: delaying package configuration, since apt-utils is not installed
  Fetched 1466 kB in 2s (608 kB/s)
  Selecting previously unselected package libunistring0:amd64.
  (Reading database ... 8857 files and directories currently installed.)
  Preparing to unpack .../0-libunistring0_0.9.6+really0.9.3-0.1_amd64.deb ...
  Unpacking libunistring0:amd64 (0.9.6+really0.9.3-0.1) ...
  Selecting previously unselected package libidn2-0:amd64.
  Preparing to unpack .../1-libidn2-0_0.16-1+deb9u1_amd64.deb ...
  Unpacking libidn2-0:amd64 (0.16-1+deb9u1) ...
  Selecting previously unselected package libpsl5:amd64.
  Preparing to unpack .../2-libpsl5_0.17.0-3_amd64.deb ...
  Unpacking libpsl5:amd64 (0.17.0-3) ...
  Selecting previously unselected package wget.
  Preparing to unpack .../3-wget_1.18-5+deb9u2_amd64.deb ...
  Unpacking wget (1.18-5+deb9u2) ...
  Selecting previously unselected package ca-certificates.
  Preparing to unpack .../4-ca-certificates_20161130+nmu1+deb9u1_all.deb ...
  Unpacking ca-certificates (20161130+nmu1+deb9u1) ...
  Selecting previously unselected package publicsuffix.
  Preparing to unpack .../5-publicsuffix_20181003.1334-0+deb9u1_all.deb ...
  Unpacking publicsuffix (20181003.1334-0+deb9u1) ...
  Processing triggers for libc-bin (2.24-11+deb9u3) ...
  Setting up publicsuffix (20181003.1334-0+deb9u1) ...
  Setting up libunistring0:amd64 (0.9.6+really0.9.3-0.1) ...
  Setting up ca-certificates (20161130+nmu1+deb9u1) ...
  debconf: unable to initialize frontend: Dialog
  debconf: (TERM is not set, so the dialog frontend is not usable.)
  debconf: falling back to frontend: Readline
  Updating certificates in /etc/ssl/certs...
  151 added, 0 removed; done.
  Setting up libidn2-0:amd64 (0.16-1+deb9u1) ...
  Setting up libpsl5:amd64 (0.17.0-3) ...
  Setting up wget (1.18-5+deb9u2) ...
  Processing triggers for libc-bin (2.24-11+deb9u3) ...
  Processing triggers for ca-certificates (20161130+nmu1+deb9u1) ...
  Updating certificates in /etc/ssl/certs...
  0 added, 0 removed; done.
  Running hooks in /etc/ca-certificates/update.d...
  done.
  Removing intermediate container 2879665e7c48
   ---> 740ce65c1344
  Step 4/16 : RUN wget https://github.com/progrium/entrykit/releases/download/v0.4.0/entrykit_0.4.0_linux_x86_64.tgz
   ---> Running in dea73313bc77
  --2018-11-25 10:38:18--  https://github.com/progrium/entrykit/releases/download/v0.4.0/entrykit_0.4.0_linux_x86_64.tgz
  Resolving github.com (github.com)... 192.30.255.113, 192.30.255.112
  Connecting to github.com (github.com)|192.30.255.113|:443... connected.
  HTTP request sent, awaiting response... 302 Found
  Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/33640915/e0224de0-c059-11e5-9b10-fbf7cc7e9fe2?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20181125%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20181125T103818Z&X-Amz-Expires=300&X-Amz-Signature=3c4428bc3152d45efa3580ef6cfed18acadafa072d2bb49101f7fc86e1f7561f&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dentrykit_0.4.0_Linux_x86_64.tgz&response-content-type=application%2Foctet-stream [following]
  --2018-11-25 10:38:18--  https://github-production-release-asset-2e65be.s3.amazonaws.com/33640915/e0224de0-c059-11e5-9b10-fbf7cc7e9fe2?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20181125%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20181125T103818Z&X-Amz-Expires=300&X-Amz-Signature=3c4428bc3152d45efa3580ef6cfed18acadafa072d2bb49101f7fc86e1f7561f&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dentrykit_0.4.0_Linux_x86_64.tgz&response-content-type=application%2Foctet-stream
  Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 54.231.82.178
  Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|54.231.82.178|:443... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 2708228 (2.6M) [application/octet-stream]
  Saving to: 'entrykit_0.4.0_linux_x86_64.tgz'

       0K .......... .......... .......... .......... ..........  1%  130K 20s
      50K .......... .......... .......... .......... ..........  3%  258K 15s
     100K .......... .......... .......... .......... ..........  5% 5.08M 10s
     150K .......... .......... .......... .......... ..........  7% 9.96M 7s
     200K .......... .......... .......... .......... ..........  9%  263K 8s
     250K .......... .......... .......... .......... .......... 11% 16.9M 6s
     300K .......... .......... .......... .......... .......... 13% 3.89M 5s
     350K .......... .......... .......... .......... .......... 15% 18.3M 4s
     400K .......... .......... .......... .......... .......... 17%  289K 5s
     450K .......... .......... .......... .......... .......... 18% 4.30M 4s
     500K .......... .......... .......... .......... .......... 20% 15.1M 4s
     550K .......... .......... .......... .......... .......... 22% 6.33M 3s
     600K .......... .......... .......... .......... .......... 24% 12.3M 3s
     650K .......... .......... .......... .......... .......... 26%  302K 3s
     700K .......... .......... .......... .......... .......... 28% 13.0M 3s
     750K .......... .......... .......... .......... .......... 30% 8.68M 3s
     800K .......... .......... .......... .......... .......... 32% 8.68M 2s
     850K .......... .......... .......... .......... .......... 34% 6.06M 2s
     900K .......... .......... .......... .......... .......... 35% 17.0M 2s
     950K .......... .......... .......... .......... .......... 37% 13.8M 2s
    1000K .......... .......... .......... .......... .......... 39% 7.66M 2s
    1050K .......... .......... .......... .......... .......... 41% 9.07M 2s
    1100K .......... .......... .......... .......... .......... 43% 8.67M 2s
    1150K .......... .......... .......... .......... .......... 45% 5.36M 1s
    1200K .......... .......... .......... .......... .......... 47% 13.5M 1s
    1250K .......... .......... .......... .......... .......... 49% 13.4M 1s
    1300K .......... .......... .......... .......... .......... 51% 8.32M 1s
    1350K .......... .......... .......... .......... .......... 52%  376K 1s
    1400K .......... .......... .......... .......... .......... 54% 22.0M 1s
    1450K .......... .......... .......... .......... .......... 56% 24.1M 1s
    1500K .......... .......... .......... .......... .......... 58% 11.8M 1s
    1550K .......... .......... .......... .......... .......... 60% 8.20M 1s
    1600K .......... .......... .......... .......... .......... 62% 12.2M 1s
    1650K .......... .......... .......... .......... .......... 64% 5.29M 1s
    1700K .......... .......... .......... .......... .......... 66% 14.0M 1s
    1750K .......... .......... .......... .......... .......... 68% 9.22M 1s
    1800K .......... .......... .......... .......... .......... 69% 13.6M 1s
    1850K .......... .......... .......... .......... .......... 71% 7.55M 1s
    1900K .......... .......... .......... .......... .......... 73% 7.35M 1s
    1950K .......... .......... .......... .......... .......... 75% 9.59M 0s
    2000K .......... .......... .......... .......... .......... 77% 8.86M 0s
    2050K .......... .......... .......... .......... .......... 79% 10.8M 0s
    2100K .......... .......... .......... .......... .......... 81% 7.26M 0s
    2150K .......... .......... .......... .......... .......... 83% 10.7M 0s
    2200K .......... .......... .......... .......... .......... 85% 11.6M 0s
    2250K .......... .......... .......... .......... .......... 86% 6.88M 0s
    2300K .......... .......... .......... .......... .......... 88% 12.3M 0s
    2350K .......... .......... .......... .......... .......... 90%  480K 0s
    2400K .......... .......... .......... .......... .......... 92% 8.43M 0s
    2450K .......... .......... .......... .......... .......... 94% 6.35M 0s
    2500K .......... .......... .......... .......... .......... 96% 11.6M 0s
    2550K .......... .......... .......... .......... .......... 98% 6.34M 0s
    2600K .......... .......... .......... .......... ....      100% 15.2M=1.6s

  2018-11-25 10:38:21 (1.62 MB/s) - 'entrykit_0.4.0_linux_x86_64.tgz' saved [2708228/2708228]

  Removing intermediate container dea73313bc77
   ---> 2c98967c1774
  Step 5/16 : RUN tar -xvzf entrykit_0.4.0_linux_x86_64.tgz
   ---> Running in ff7abe00309c
  entrykit
  Removing intermediate container ff7abe00309c
   ---> addb23c07603
  Step 6/16 : RUN rm entrykit_0.4.0_linux_x86_64.tgz
   ---> Running in 22f15256cab4
  Removing intermediate container 22f15256cab4
   ---> 3e734f4463ea
  Step 7/16 : RUN mv entrykit /usr/local/bin/
   ---> Running in 8282d8355551
  Removing intermediate container 8282d8355551
   ---> 15c3a13a9dbf
  Step 8/16 : RUN entrykit --symlink
   ---> Running in ce5b7cf07987
  Creating symlink /usr/local/bin/entrykit ...
  Creating symlink /usr/local/bin/codep ...
  Creating symlink /usr/local/bin/prehook ...
  Creating symlink /usr/local/bin/render ...
  Creating symlink /usr/local/bin/switch ...
  Removing intermediate container ce5b7cf07987
   ---> 4db462ea1227
  Step 9/16 : COPY add-server-id.sh /usr/local/bin/
   ---> 2302bd5ce059
  Step 10/16 : COPY etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/
   ---> 22546372bdd1
  Step 11/16 : COPY etc/mysql/conf.d/mysql.cnf /etc/mysql/conf.d/
   ---> cefe14b8ebd9
  Step 12/16 : COPY prepare.sh /docker-entrypoint-initdb.d
   ---> 1c5e3d21135f
  Step 13/16 : COPY init-data.sh /usr/local/bin/
   ---> 9063c3add476
  Step 14/16 : COPY sql /sql
   ---> bc4b49fcff4a
  Step 15/16 : ENTRYPOINT [   "prehook",     "add-server-id.sh",     "--",   "docker-entrypoint.sh" ]
   ---> Running in a719ec77bf52
  Removing intermediate container a719ec77bf52
   ---> 81427966d5ba
  Step 16/16 : CMD ["mysqld"]
   ---> Running in f7e703b2ecee
  Removing intermediate container f7e703b2ecee
   ---> e8a9a539cdd7
  Successfully built e8a9a539cdd7
  Successfully tagged ch04/tododb:latest


タグ名をつける。

.. code-block:: bash

  $ docker image tag ch04/tododb:latest localhost:5000/ch04/tododb:latest


Swarm クラスタの worker ノードで利用できるように、 ``localhost:5000/ch04/tododb:latest`` という名前で registry に push しておく。

.. code-block:: bash

  $ docker image push localhost:5000/ch04/tododb:latest
  The push refers to repository [localhost:5000/ch04/tododb]
  7145c9f07816: Preparing
  f31f10668bad: Preparing
  18b21f6c50ce: Preparing
  2ad5fd0a3131: Preparing
  1f22bafd4a4b: Preparing
  f208eba8785e: Preparing
  eb3efcf4de85: Preparing
  77cc7b793ed5: Preparing
  09c90391a311: Preparing
  e9651980f89e: Preparing
  1e9b0de5a957: Preparing
  9adbc7c066aa: Preparing
  e618193140e0: Preparing
  0d954c604c76: Preparing
  ceb15396dc26: Preparing
  347571a8da20: Pushed
  ea66b8e6103f: Pushed
  3d4164460bf0: Pushed
  783b13a988e3: Pushed
  2566141f200b: Pushed
  0ad177796f33: Pushed
  0f1205f1cd43: Pushed
  a588c986cf97: Pushed
  ef68f6734aa4: Pushed
  latest: digest: sha256:d2108914ffdc5e80d08fa4e07067c7872dc427c6f0ef7df208de05e3f5fecd91 size: 5334


4.2.6 Swarm 上で Master/Slave サービスを実行する
-------------------------------------------------

``todo-mysql.yml`` を作成する。

.. code-block:: yaml

  version: "3"

  services:
    master:
      image: registry:5000/ch04/tododb:latest
      deploy:
        replicas: 1
        placement:
          constraints: [node.role != manager]  # manager ノード以外に配置する
      environment:
        MYSQL_ROOT_PASSWORD: gihyo
        MYSQL_DATABASE: tododb
        MYSQL_USER: gihyo
        MYSQL_PASSWORD: gihyo
        MYSQL_MASTER: "true"
      networks:
        - todoapp

    slave:
      image: registry:5000/ch04/tododb:latest
      deploy:
        replicas: 2
        placement:
          constraints: [node.role != manager]  # manager ノード以外に配置する
      depends_on:
        - master
      environment:
        MYSQL_MASTER_HOST: master
        MYSQL_ROOT_PASSWORD: gihyo
        MYSQL_DATABASE: tododb
        MYSQL_USER: gihyo
        MYSQL_PASSWORD: gihyo
        MYSQL_REPL_USER: repl
        MYSQL_REPL_PASSWORD: gihyo
      networks:
        - todoapp

  networks:
    todoapp:            # overlay ネットワーク
      external: true    # 各 Service をこのネットワークに所属させる


Swarm によるデプロイ
^^^^^^^^^^^^^^^^^^^^

manager コンテナに、 ``todo-mysql.yml`` を ``todo_mysql`` スタックとしてデプロイする。

.. code-block:: bash

  $ docker container exec -it manager docker stack deploy -c /stack/todo-mysql.yml todo-mysql
  Creating service todo-mysql_master
  Creating service todo-mysql_slave

  # デプロイされているサービス一覧を確認する。
  $ docker container exec -it manager docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
  wx74zcnjz77k        todo-mysql_master   replicated          1/1                 registry:5000/ch04/tododb:latest
  3r61ptmsixr5        todo-mysql_slave    replicated          2/2                 registry:5000/ch04/tododb:latest


4.2.7 MySQL コンテナを確認し、初期データを投入する
--------------------------------------------------

Master コンテナが Swarm のどのノードに配置されたか調べる。

- ノードの ID と Task ID がわかれば ``docker container exec`` を多段で実行することで目的のコンテナの中にはいることができる。

.. code-block:: console

  $ docker container exec -it manager docker service ps todo-mysql_master --no-trunc --filter "desired-state=running"
  ID                          NAME                  IMAGE                                                                                                      NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
  pitobepehy19jm3v7pfe2cavo   todo-mysql_master.1   registry:5000/ch04/tododb:latest@sha256:d2108914ffdc5e80d08fa4e07067c7872dc427c6f0ef7df208de05e3f5fecd91   7190447651de        Running             Running 9 minutes ago


ノードの ID と Task ID を調べるのと同時に、 ``--format`` を使って多段用のコマンドも作っちゃう。

.. code-block:: console

  $ docker container exec -it manager \
    docker service ps todo-mysql_master \
    --no-trunc \
    --filter "desired-state=running" \
    --format "docker container exec -it {{.Node}} docker container exec -it {{.Name}}.{{.ID}} bash"


master コンテナで ``init-data.sh`` を実行してテーブルとデータを作成します。

.. code-block:: bash

  # 初期データ投入スクリプトを実行する
  $ docker container exec -it 7190447651de docker container exec -it todo-mysql_master.1.pitobepehy19jm3v7pfe2cavo init-data.sh

  # データが正常に投入できたか確認する
  $ docker container exec -it 7190447651de docker container exec -it todo-mysql_master.1.pitobepehy19jm3v7pfe2cavo \
  >   mysql -u gihyo -pgihyo tododb
  mysql: [Warning] Using a password on the command line interface can be insecure.
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A

  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 16
  Server version: 5.7.24-log MySQL Community Server (GPL)

  Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql> SELECT * FROM todo LIMIT 1\G
  *************************** 1. row ***************************
       id: 1
    title: MySQLのDockerイメージを作成する
  content: MySQLのMaster、Slaveそれぞれで利用できるように、環境変数で役割を制御できるMySQLイメージを作成する
   status: DONE
  created: 2018-11-25 11:34:16
  updated: 2018-11-25 11:34:16
  1 row in set (0.00 sec)


Master に登録したデータが Slave にも反映されていることを確認する。

.. code-block:: bash

  # ノードの ID と Task ID を調べる
  $ docker container exec -it manager \
    docker service ps todo-mysql_slave \
    --no-trunc \
    --filter "desired-state=running" \
    --format "docker container exec -it {{.Node}} docker container exec -it {{.Name}}.{{.ID}} bash"
    docker container exec -it 272227e51007 docker container exec -it todo-mysql_slave.1.xd9oxztk61ogea9duail6yhv0 bash
    docker container exec -it cfffba103c8c docker container exec -it todo-mysql_slave.2.rk8cm47671v1ah04s9teajft1 bash

  # データが反映できたことを確認する (slave1)
  $ docker container exec -it 272227e51007 docker container exec -it todo-mysql_slave.1.xd9oxztk61ogea9duail6yhv0 mysql -u gihyo -pgihyo tododb
  mysql: [Warning] Using a password on the command line interface can be insecure.
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A

  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 4
  Server version: 5.7.24-log MySQL Community Server (GPL)

  Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql> SELECT * FROM todo LIMIT 1\G
  *************************** 1. row ***************************
       id: 1
    title: MySQLのDockerイメージを作成する
  content: MySQLのMaster、Slaveそれぞれで利用できるように、環境変数で役割を制御できるMySQLイメージを作成する
   status: DONE
  created: 2018-11-25 11:34:16
  updated: 2018-11-25 11:34:16
  1 row in set (0.00 sec)

  mysql> exit
  Bye

  # データが反映できたことを確認する (slave2)
  $ docker container exec -it cfffba103c8c docker container exec -it todo-mysql_slave.2.rk8cm47671v1ah04s9teajft1 mysql -u gihyo -pgihyo tododb
  mysql: [Warning] Using a password on the command line interface can be insecure.
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A

  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 4
  Server version: 5.7.24-log MySQL Community Server (GPL)

  Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql> SELECT * FROM todo LIMIT 1\G
  *************************** 1. row ***************************
       id: 1
    title: MySQLのDockerイメージを作成する
  content: MySQLのMaster、Slaveそれぞれで利用できるように、環境変数で役割を制御できるMySQLイメージを作成する
   status: DONE
  created: 2018-11-25 11:34:16
  updated: 2018-11-25 11:34:16
  1 row in set (0.01 sec)


4.3 API Service の構築
=======================


