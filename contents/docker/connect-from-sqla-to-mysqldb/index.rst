.. article::
   :title: Docker Compose で MySQL DB をつくり Compose 外 (非 docker) の SQLAlchemy から接続する
   :date: 2019-03-24
   :category: docker
   :tags:
   :canonical_url: /docker/connect-from-sqla-to-mysqldb/
   :draft: false


=====================================================================================
Docker Compose で MySQL DB をつくり Compose 外 (非 docker) の SQLAlchemy から接続する
=====================================================================================

.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス・ガイド
=====================

SQLAlchemy
-----------
- `SQLAlchemy 1.3 Documentation: MySQL <https://docs.sqlalchemy.org/en/latest/dialects/mysql.html>`_

Docker Hub
-----------
- `mysql Docker Official Images <https://hub.docker.com/_/mysql>`_

PyPI
----
- `mysqlclient <https://pypi.org/project/mysqlclient/>`_
- `PyMySQL <https://pypi.org/project/PyMySQL/>`_
- `mysql-connector-python <https://pypi.org/project/mysql-connector-python/>`_


MySQL DB をつくる
==================

1. 準備
--------

ディレクトリ構成
^^^^^^^^^^^^^^^^^

.. code-block:: shell

  $ tree mmm
  mmm
  ├── Dockerfile-mysql
  └── docker-compose.yml


設定ファイル
^^^^^^^^^^^^^^^^^

- Dockerfile-mysql

  .. code-block:: docker

    FROM mysql:5.7.12
    # `5.7.12` は自分の使いたいイメージのタグ
    RUN apt-get clean && apt-get update && apt-get install -y locales locales-all vim
    # ↑のイメージそのままだと vi も vim も入っていなくて不便だったので vim も入れておく
    RUN locale-gen ja_JP.UTF-8
    ENV LANG ja_JP.UTF-8
    ENV LANGUAGE ja_JP:en
    ENV LC_ALL ja_JP.UTF-8
    RUN ln -sf /usr/share/zoneinfo/Japan /etc/localtime


- docker-compose.yml

  .. code-block:: yaml

    version: '3'

    services:
      db:
        container_name: mmm_db
        build:
          context: .
          dockerfile: Dockerfile-mysql
        # 文字コードの設定をしておく
        command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
        restart: always
        volumes:
          - "db-data:/var/lib/mysql"
        environment:
          MYSQL_ROOT_PASSWORD: mmm
          MYSQL_DATABASE: mmm
          MYSQL_USER: mmm
          MYSQL_PASSWORD: mmm
        ports:
          - "3306:3306"  # 大事。ここを書いておかないと、 compose の外から繋げない。

    volumes:
      db-data:


2. 起動
--------

.. code-block:: shell

  # mmm 直下で実行する
  $ docker-compose up



SQLAlchemy の設定
==================

mysqlclient
-----------

接続文字列
^^^^^^^^^^

.. code-block:: python

  mysql+mysqldb://mmm:mmm@127.0.0.1:3306/mmm?charset=utf8mb4
  # mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>

事前準備
^^^^^^^^
1. Python3 他のインストール

  - sudo apt-get install python3-dev default-libmysqlclient-dev ＃Debian / Ubuntu
  - sudo yum install python3-devel mysql-devel ＃Red Hat / CentOS
  - brew install mysql-connector-c # macOS (Homebrew) (Currently, it has bug. See below)

      - macOS の場合はバグあるらしくちょっと小細工が必要

        - ``/usr/local/bin/mysql_config`` を編集しないといけない (PyPI の Project description, GitHub の README に書いてある)
        - ``mysql_config`` は ``$ which mysql_config`` で探せる

2. mysqlclient のインストール

    .. code-block:: bash

      pip install mysqlclient


PyMySQL
--------

接続文字列
^^^^^^^^^^

.. code-block:: python

  mysql+pymysql://mmm:mmm@127.0.0.1:3306/mmm?charset=utf8mb4
  # mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]


事前準備
^^^^^^^^

PyMySQL のインストール

  .. code-block:: bash

    pip install PyMySQL


mysql-connector-python
----------------------

接続文字列
^^^^^^^^^^

.. code-block:: python

  mysql+mysqlconnector://mmm:mmm@127.0.0.1:3306/mmm?charset=utf8mb4"
  # mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>


事前準備
^^^^^^^^
mysql-connector-python のインストール

  .. code-block:: bash

    pip install mysql-connector-python


使ったメモ
===========

.. list-table::
  :widths:  auto
  :header-rows: 1
  :stub-columns: 1

  * -
    - mysqlclient
    - PyMySQL
    - mysql-connector-python
  * - 準備 ※1
    - ただの Python3 だけでは済まない、 ``python3-dev`` をインストールする必要がある

        - ``python3`` パッケージ (実態は ``python3.7`` とか) は実行に必要なものだけが入る
        - ``Python.h`` などのヘッダファイルや ``python3.7.a`` などのスタティックリンクライブラリは入っていない
        - これらは、 ``python3-dev`` (``python3.7-dev``) でインストールされる。実行に必要がないため別れている。
        - mysqlclient は C拡張を含んでるのでビルドする必要がある
        - ビルドには ``Python.h`` などのヘッダファイルが必要
        - 昨今は C拡張であってもビルド済の wheel が pypi にあがってたりしてインストール時にビルドが必要ないものも増えているが、 mysqlclient は SSL のリンクの都合上 wheel を提供していないよう

    - ただの Python3 だけで済む

      - PyMySQL は C拡張を含まずピュア Python なのでビルドする必要がない
      - この場合 SSL はpythonをインストールしたときにリンクしたものを使う

    - ただの Python3 だけで済む
  * - [Note] Aborted connection ※2
    - ``log_warnings = 2`` でも出ない
    - ``log_warnings = 2`` だと出る, 1 だと出ない
    - ``log_warnings = 2`` だと出る, 1 だと出ない
  * - SQLAlchemy のおし具合
    - - `mysqlclient supports Python 2 and Python 3 and is very stable.`
      - 一番おすすめに見える (個人の感想)
    - - `The pymysql DBAPI is a pure Python port of the MySQL-python (MySQLdb) driver, and targets 100% compatibility.`
      - 二番目におすすめに見える (個人の感想)
    - - 特記事項なし
      - 付け加えて言いたいことがないようなので、ふつうかな (個人の感想)
  * - わたしの感想
    - - ただの Python3 だけでは済まないのが手軽感減少、Mac だとけっこうめんどう。どうにかしてほしい。
      - でも ``Aborted connection`` が出ないのはなるほどと思った
      - Django もこれをおすすめしていたので、できればこれが良いが、インストールのところがどうしてもひっかかる。
    - install が手軽でよい。 ``Aborted connection`` はあまり気にしなくて良さそうでもあるし、 SQLA さんも二番目におすすめしている (空想) のでこれがいいかなあ。
    - install が手軽でよい。ほかはとくになし。
  * - aodag さんに教えてもらったことメモ
    -
    - **sqlalchemy で使うなら pymysql 使っとけ（断言）**
    -


※ 1.
------
``python3-dev`` が必要 or 不要な理由も aodag さんに教えていただきました。ありがとうございました。


※ 2. [Note] Aborted connection ...
------------------------------------
- `B.6.2.10 Communication Errors and Aborted Connections <https://dev.mysql.com/doc/refman/5.7/en/communication-errors.html>`_


  .. code-block:: bash

      db_1  | 2019-03-24T06:38:23.691896Z 2 [Note] Aborted connection 2 to db: 'mmm' user: 'mmm' host: '172.27.0.1' (Got an error reading communication packets)

  - クライアントの接続方法とか切断方法に何か問題があるらしい
  - わたしの場合、同じコードでもドライバーによって出たり出なかったりする
  - ログレベルを下げると出なくなる
  - 同じ事象のひとが世界中にけっこういる
  - このログが出ていても、(ワーニングログがたくさん出ること以外に)「困った!」というひとはあまりいなそう


参考ページ
^^^^^^^^^^^
- `MySQLで「Got an error reading communication packets」というエラーが出力される原因と対策 <https://weblabo.oscasierra.net/mysql-error-reading-communication-packets/>`_
- `[MariaDB] Aborted connectionのワーニング対応に大いにハマる・・ <https://qiita.com/hit/items/da50428ca4b4162162a8>`_

などなど... ありがとうございました。
