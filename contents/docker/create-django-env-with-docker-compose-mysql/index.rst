.. article::
   :title: Docker Compose で Django/MySQL 環境をつくる
   :date: 2019-01-05
   :category: docker
   :tags:
   :canonical_url: /docker/create-django-env-with-docker-compose-mysql/
   :draft: false


===========================================
Docker Compose で Django/MySQL 環境をつくる
===========================================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス・ガイド
====================
- `Compose file version 3 reference <https://docs.docker.com/compose/compose-file/>`_

- `Docker Hub の MySQL 公式イメージぺージ <https://hub.docker.com/_/mysql/>`_

  - ``docker-compose`` ファイルの書き方見本とか、 Environment Variables の一覧とかが載っていた

- `MySQL 8.0 Reference Manual <https://dev.mysql.com/doc/refman/8.0/en/>`_

  - MySQL 自体のリファレンス
  - `MySQL Program Environment Variables <https://dev.mysql.com/doc/refman/8.0/en/environment-variables.html>`_ :
    ``docker-compose`` ファイルの ``environment`` 部分に書けるやつを探すときに使った
  - `Server Command Options <https://dev.mysql.com/doc/refman/8.0/en/server-options.html>`_ :
    ``mysql.cnf`` に書けるやつ、 ``docker-compose`` ファイルの ``command`` 部分に書けるやつを探すときに使った
  - `Server System Variables <https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html>`_ :
    ``mysql.cnf`` に書けるやつを探すときに使った

- `PyPI の mysql-connector-python のページ <https://pypi.org/project/mysql-connector-python/>`_

  - MySQL の driver。これが Oracle 純正のものらしい。

- `MySQL Connector/Python Developer Guide <https://dev.mysql.com/doc/connector-python/en/>`_

  - mysql-connector-python のガイドページ。使い方や見本がいろいろ載っている。

- `PyPI の mysqlclient のページ <https://pypi.org/project/mysqlclient/>`_

- `Django で MySQL を使うときは見てねページ <https://docs.djangoproject.com/ja/2.1/ref/databases/#mysql-notes>`_

  - Django のほうの説明を読んだら、 ``mysqlclient`` がおすすめ、かつ、 `Django requires mysqlclient 1.3.7 or later.` と書いてある。
  - とりあえず ``mysqlclient`` と ``mysql-connector-python`` を両方入れてみる。


環境構築
========

最初の1回だけ実行
-----------------

1. web のほうの docker image をビルドして、 startproject する。

    .. code-block:: bash

      $ docker-compose --pull=true run web django-admin.py startproject fu .

    - ``--pull=true`` : FROM に指定したベースイメージがローカルにすでに存在していても、改めて Docker Hub に取得しにいく
    - ↑今やったら使えなかった。最新イメージを再取得するには、どうすればいいんだろう。

2. settings に MySQL の設定を書き足す

    .. code-block:: python

      DATABASES = {
          'default': {
              'ENGINE': 'django.db.backends.mysql',
              'NAME': 'fu',
              'USER': 'fu',
              'PASSWORD': 'fu',
              'HOST': 'db',
              'PORT': 3306,
          }
      }

3. docker-compose up して mysql コンテナ用のイメージをビルド、 mysql・web コンテナを起動する

    .. code-block:: bash

      $ docker-compose up


2回目以降
---------
1. docker-compose up して mysql・web コンテナを起動する。

    .. code-block:: bash

      $ docker-compose up

2. 安全にシャットダウン。コンテナの停止と削除。

    .. code-block:: bash

      $ docker-compose down


設定ファイル
------------
- fu/docker-compose.yml

  .. code-block:: yaml

    version: '3'

    services:
      db:
        container_name: fu_db
        build:
          context: .
          dockerfile: Dockerfile-mysql
        restart: always
        volumes:
          - "db-data:/var/lib/mysql"
          # mysql のカスタム設定ファイルもホスト <-> コンテナ間で同期しておく (編集するのに便利だから)
          - "./mysql/conf.d:/etc/mysql/conf.d"
        environment:
          MYSQL_ROOT_PASSWORD: fu
          MYSQL_DATABASE: fu
          MYSQL_USER: fu
          MYSQL_PASSWORD: fu

      web:
        container_name: fu_web
        build:
          context: .
          dockerfile: Dockerfile-web
        command: python3 manage.py runserver 0.0.0.0:8000 --settings=settings._
        volumes:
          - .:/code
        ports:
          - "3236:8000"
        depends_on:
          - db

    volumes:
      db-data:


- fu/Dockerfile-web

  .. code-block:: docker

    FROM python:3
    ENV PYTHONUNBUFFERED 1
    RUN mkdir /code
    WORKDIR /code
    ADD requirements.txt /code/
    RUN pip install -r requirements.txt
    ADD . /code/


- fu/Dockerfile-mysql

  .. code-block:: docker

    FROM mysql:latest
    # locales をインストールする
    RUN apt-get clean && apt-get update && apt-get install -y locales locales-all
    # locale 定義ファイルをコンパイルする
    RUN locale-gen ja_JP.UTF-8
    # 日本語を設定する TODO: もしかして LANG だけでいいのかも...?
    ENV LANG ja_JP.UTF-8
    ENV LANGUAGE ja_JP:en
    ENV LC_ALL ja_JP.UTF-8
    # タイムゾーンに日本を設定する
    RUN ln -sf /usr/share/zoneinfo/Japan /etc/localtime


- fu/requirements.txt

  .. code-block:: python

    Django>=1.11
    mysqlclient>=1.3.7
    mysql-connector-python


- fu/mysql/conf.d/mysql.cnf

  .. code-block:: cfg

    [mysqld]  # mysqlサーバーの設定
    default-authentication-plugin=mysql_native_password
    # Collation (文字照合順) の設定: https://mysqlserverteam.com/mysql-8-0-kana-sensitive-collation-for-japanese-ja_jp/
    collation-server=utf8mb4_ja_0900_as_cs_ks
    # サーバー側の文字コードの設定: "Default Value (>= 8.0.1)	utf8mb4" だけれどもなんとなく念のため指定
    character-set-server=utf8mb4
    # time_zone の設定: https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_default-time-zone
    default-time-zone='Asia/Tokyo'

    [client]  # mysqlクライアントの設定
    # クライアント側の文字コードの設定
    # utf8mb4: A UTF-8 encoding of the Unicode character set using one to four bytes per character.
    default-character-set=utf8mb4


  - MySQL デフォルトの設定ファイルは ``/etc/mysql/my.cnf`` の模様。 MySQL 公式イメージからコンテナを作成すると、初期状態で ``/etc/mysql/my.cnf`` 中に

    .. code-block:: cfg

      # Custom config should go here
      !includedir /etc/mysql/conf.d/

    と書いてあるので、言に従い自分用設定は ``/etc/mysql/conf.d/`` に記述した。


ディレクトリ構成
----------------

.. code-block:: shell

  $ tree fu

  fu
  ├── Dockerfile-mysql
  ├── Dockerfile-web
  ├── docker-compose.yml
  ├── fu
  │   ├── __init__.py
  │   ├── urls.py
  │   └── wsgi.py
  ├── manage.py
  ├── mysql
  │   └── conf.d
  │       └── mysql.cnf
  ├── requirements.txt
  └── settings
      └── _.py


使い方メモ
==========

イメージビルドからやり直したい
------------------------------
こんなときは...

- requirements.txt にライブラリを書き足したとき
- Dockerfile を更新したとき

.. code-block:: bash

  $ docker-compose build
  $ docker-compose up

もしくは、

.. code-block:: bash

  $ docker-compose up --build

volume は消さない限り再利用される
---------------------------------

.. code-block:: yaml

  # fu/docker-compose.yml

  volumes:
    db-data:


.. code-block:: bash

  $ docker volume ls
  DRIVER              VOLUME NAME
  local               fu_db-data  # こういう名前がついている


web のほうのコンテナだけ再起動したいとき
----------------------------------------
.. code-block:: bash

  $ docker container restart fu_web


web のほうのコンテナに入る
--------------------------
VM の中に入るのと同じような気持ち。

.. code-block:: bash

  $ docker container exec -it fu_web bash
  root@eb1ab9a6dbdb:/code#

  # 現在のロケール設定を確認する
  root@eb1ab9a6dbdb:/code# locale
  LANG=C.UTF-8
  LANGUAGE=
  LC_CTYPE="C.UTF-8"
  LC_NUMERIC="C.UTF-8"
  LC_TIME="C.UTF-8"
  LC_COLLATE="C.UTF-8"
  LC_MONETARY="C.UTF-8"
  LC_MESSAGES="C.UTF-8"
  LC_PAPER="C.UTF-8"
  LC_NAME="C.UTF-8"
  LC_ADDRESS="C.UTF-8"
  LC_TELEPHONE="C.UTF-8"
  LC_MEASUREMENT="C.UTF-8"
  LC_IDENTIFICATION="C.UTF-8"
  LC_ALL=


mysql のほうのコンテナに入る
------------------------------

.. code-block:: bash

  # mysql のコンテナに入る
  $ docker container exec -it fu_db bash

  # 現在のロケール設定を確認する
  root@dcc8b4d2cd21:/# locale
  LANG=ja_JP.UTF-8
  LANGUAGE=ja_JP:en
  LC_CTYPE="ja_JP.UTF-8"
  LC_NUMERIC="ja_JP.UTF-8"
  LC_TIME="ja_JP.UTF-8"
  LC_COLLATE="ja_JP.UTF-8"
  LC_MONETARY="ja_JP.UTF-8"
  LC_MESSAGES="ja_JP.UTF-8"
  LC_PAPER="ja_JP.UTF-8"
  LC_NAME="ja_JP.UTF-8"
  LC_ADDRESS="ja_JP.UTF-8"
  LC_TELEPHONE="ja_JP.UTF-8"
  LC_MEASUREMENT="ja_JP.UTF-8"
  LC_IDENTIFICATION="ja_JP.UTF-8"
  LC_ALL=ja_JP.UTF-8


mysql に入る
------------

.. code-block:: bash

  # mysql のコンテナに入る
  $ docker container exec -it fu_db bash
  root@6ba1661872de:/#

  # mysql にログインする
  $ mysql -h localhost -u fu -p fu

  Enter password: (compose ファイルに定義したパスワード)

  # 現在の Character Sets 設定を表示する
  mysql> SHOW VARIABLES LIKE "char%";
  +--------------------------+--------------------------------+
  | Variable_name            | Value                          |
  +--------------------------+--------------------------------+
  | character_set_client     | utf8mb4                        |
  | character_set_connection | utf8mb4                        |
  | character_set_database   | utf8mb4                        |
  | character_set_filesystem | binary                         |
  | character_set_results    | utf8mb4                        |
  | character_set_server     | utf8mb4                        |
  | character_set_system     | utf8                           |
  | character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
  +--------------------------+--------------------------------+
  8 rows in set (0.01 sec)

  # 現在のタイムゾーン設定を表示する
  mysql> SHOW VARIABLES LIKE '%time_zone%';
  +------------------+------------+
  | Variable_name    | Value      |
  +------------------+------------+
  | system_time_zone | JST        |  # MySQL が動いているコンテナ (Debian) のシステムタイムゾーン、Dockerfile-mysql で設定した
  | time_zone        | Asia/Tokyo |  # mysql.cnf で設定した MySQL サーバー側のタイムゾーン
  +------------------+------------+
  2 rows in set (0.01 sec)

  mysql> status
  --------------
  mysql  Ver 8.0.13 for Linux on x86_64 (MySQL Community Server - GPL)

  Connection id:		11
  Current database:	fu
  Current user:		fu@localhost
  SSL:			Not in use
  Current pager:		stdout
  Using outfile:		''
  Using delimiter:	;
  Server version:		8.0.13 MySQL Community Server - GPL
  Protocol version:	10
  Connection:		Localhost via UNIX socket
  Server characterset:	utf8mb4
  Db     characterset:	utf8mb4
  Client characterset:	utf8mb4
  Conn.  characterset:	utf8mb4
  UNIX socket:		/var/run/mysqld/mysqld.sock
  Uptime:			2 hours 9 min 49 sec

  Threads: 3  Questions: 40  Slow queries: 0  Opens: 189  Flush tables: 2  Open tables: 163  Queries per second avg: 0.005
  --------------

TODO
====
web コンテナのほうの locale と タイムゾーンは db コンテナと合わせなくて大丈夫なんだろうか...
