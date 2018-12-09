.. article::
   :title: Docker Compose で Volumes をつかう
   :date: 2018-12-02
   :category: docker
   :tags:
   :canonical_url: /docker/use-volumes-with-docker-compose/
   :draft: false


================================
Docker Compose で Volumes を使う
================================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス
============
- `Use volumes <https://docs.docker.com/storage/volumes/>`_
- `Create and manage volumes <https://docs.docker.com/storage/volumes/#create-and-manage-volumes>`_
- https://docs.docker.com/compose/compose-file/#external
- https://docs.docker.com/compose/compose-file/#environment
- http://www.postgresqltutorial.com/psql-commands/


手順
====
``web`` のイメージをビルド...するところは、
:jinja:`{{ content.link_to("../create-django-env-with-docker-compose/index.rst") }}`
を参照のこと。


1. volume を作成する
--------------------
``db-data`` という名前の volume を作成する。

.. code-block:: shell

  $ docker volume create db-data
  db-data


- volumes を一覧表示する。

  .. code-block:: shell

    $ docker volume ls
    DRIVER              VOLUME NAME
    local               db-data


- volume を調べる。

  .. code-block:: shell

    $ docker volume inspect db-data
    [
        {
            "CreatedAt": "2018-12-02T13:09:48Z",
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "/var/lib/docker/volumes/db-data/_data",
            "Name": "db-data",
            "Options": {},
            "Scope": "local"
        }
    ]


- 今作った ``volume`` はこの絵↓の真ん中のものである。

  .. figure :: use-volumes.png

  https://docs.docker.com/storage/volumes/


2. ``docker-compose.yml`` に作成した volume を指定する
------------------------------------------------------

.. code-block:: yaml

  version: '3'

  services:
    db:
      image: postgres:latest
      volumes:
        # コンテナの `/var/lib/postgresql/data` を、volume `db-data` にマウントする。
        - "db-data:/var/lib/postgresql/data"
      # 環境変数を追加する
      environment:
        # postgres のパスワードを設定する
        - POSTGRES_PASSWORD=postgres
    web:
      build:
        context: .
        dockerfile: Dockerfile-web
      command: python3 manage.py runserver 0.0.0.0:8000
      volumes:
        - .:/code
      ports:
        - "3236:8000"
      depends_on:
        - db

  # volumes を定義する
  volumes:
    # volume の名前を指定
    db-data:
      # Compose の外ですでに作成済みの volume を指定する場合は ture を設定する。
      # そうすると、 docker-compose up 時に Compose は volume を作成しようとしません。
      # かつ、指定した volume が存在しないとエラーを raise します。
      external: true


- ``external: true`` を書かないと、 ``$ docker-compose up`` 時に 1. で作った ``db-data`` とは別に
  ``{プロジェクト名}_db-data`` という感じの名前の volume が作られて、そちらが使われる。
- それでもまあかまわないけれど。今のところは。


3. ``settings.py`` に DATABASE を定義する
-----------------------------------------

.. code-block:: python

  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.postgresql',
          'NAME': 'postgres',
          'USER': 'postgres',
          'PASSWORD': 'postgres',  # パスワードを追加した
          'HOST': 'db',
          'PORT': 5432,
      }
  }


4. Docker コンテナを実行する
----------------------------

.. code-block:: bash

  $ docker-compose up


5. DB コンテナのなかに入って、テーブルを作成 & データを挿入する
-------------------------------------------------------------------

1). ``db`` コンテナのなかに入る

  .. code-block:: bash

    # コンテナをシェル経由で操作する
    $ docker container exec -it fffff_db_1 sh


2). postgres に接続する

  .. code-block:: postgres

    -- postgres に接続するその１
    # psql -U postgres -h 127.0.0.1 -p 5432 postgres
    psql (11.1 (Debian 11.1-1.pgdg90+1))
    Type "help" for help.

    postgres=#

    -- postgres に接続するその２
    # psql -d postgres -U  postgres -W
    Password:
    psql (11.1 (Debian 11.1-1.pgdg90+1))
    Type "help" for help.

    postgres=#

3). テーブルを作成し、データを挿入する

  .. code-block:: postgres

    CREATE TABLE fruits(
       id SERIAL PRIMARY KEY,
       name VARCHAR NOT NULL
    );
    INSERT INTO fruits(name) VALUES('orange');
    INSERT INTO fruits(id,name) VALUES(DEFAULT,'apple');

    postgres=# \q


6. 安全に shutdown する
-----------------------
コンテナは停止・削除される。

.. code-block:: bash

  $ docker-compose down


7. 再度 ``docker-compose up`` して、先ほど登録したテーブル & データを見てみる
-------------------------------------------------------------------------------

先ほど作成したテーブルとデータがありました。

.. code-block:: console

  $ docker-compose up
  $ docker container exec -it fffff_db_1 sh


.. code-block:: postgres

  # psql -U postgres -h 127.0.0.1 -p 5432 postgres
  psql (11.1 (Debian 11.1-1.pgdg90+1))b
  Type "help" for help.

  postgres=# \dt
           List of relations
   Schema |  Name  | Type  |  Owner
  --------+--------+-------+----------
   public | fruits | table | postgres
  (1 row)

  postgres=# SELECT * FROM fruits;
   id |  name
  ----+--------
    1 | orange
    2 | apple
  (2 rows)
