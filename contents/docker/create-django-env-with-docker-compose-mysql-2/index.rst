.. article::
   :title: Docker Compose で Django/MySQL 環境をつくる その2
   :date: 2019-01-20
   :category: docker
   :tags:
   :canonical_url: /docker/create-django-env-with-docker-compose-mysql-2/
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


リファレンス・ガイド・参考書籍
==============================
- :jinja:`{{ content.link_to("../create-django-env-with-docker-compose-mysql/index.rst") }}` と同じ
- 現場で使える Django の教科書《基礎編》


環境構築
========

1. 準備
-------

ディレクトリ構成
^^^^^^^^^^^^^^^^

.. code-block:: shell

  $ tree fufu
  fufu
  ├── Dockerfile-mysql
  ├── Dockerfile-web
  ├── docker-compose.yml
  ├── mysql
  │   └── conf.d
  │       └── mysql.cnf
  └── requirements.txt

設定ファイル
^^^^^^^^^^^^^

- Dockerfile-web

  .. code-block:: docker

    FROM python:3
    ENV PYTHONUNBUFFERED 1
    RUN mkdir /code
    WORKDIR /code
    ADD requirements.txt /code/
    RUN pip install -r requirements.txt
    ADD . /code/
    RUN mkdir apps  # アプリケーションは `apps` ディレクトリ配下に作りたかったので、追加
    WORKDIR apps  # 同上


- docker-compose.yml

  .. code-block:: yaml

    services:
      db:
        container_name: fufu_db
        build:
          context: .
          dockerfile: Dockerfile-mysql
        restart: always
        volumes:
          - "db-data:/var/lib/mysql"
          - "./mysql/conf.d:/etc/mysql/conf.d"
        environment:
          MYSQL_ROOT_PASSWORD: fufu
          MYSQL_DATABASE: fufu
          MYSQL_USER: fufu
          MYSQL_PASSWORD: fufu

      web:
        container_name: fufu_web
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


- mysql.cnf

  .. code-block:: cfg

    [mysqld]  # *** mysqlサーバーの設定 ***************
    # https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password
    # https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password-compatible-connectors
    # https://dev.mysql.com/doc/mysqld-version-reference/en/optvar-changes-8-0.html
    # [環境構築メモ ※1 参照のこと]
    default-authentication-plugin=mysql_native_password

    # デフォルト状態で /etc/mysql/conf.d/docker.cnf に定義されているやつをコピーしておく
    # [環境構築メモ ※2 参照のこと]
    skip-host-cache
    skip-name-resolve

    # https://mysqlserverteam.com/mysql-8-0-kana-sensitive-collation-for-japanese-ja_jp/
    collation-server=utf8mb4_ja_0900_as_cs_ks
    character-set-server=utf8mb4
    # https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_default-time-zone
    default-time-zone='Asia/Tokyo'

    [client]  # *** mysqlクライアントの設定 ************
    default-character-set=utf8mb4


- あとの設定ファイルは :jinja:`{{ content.link_to("../create-django-env-with-docker-compose-mysql/index.rst") }}` と同じ


2. 最初の1回だけ実行
--------------------

web, db の docker image をビルド -> web で startproject する。 (ログを良く見ると、たぶん db コンテナの起動もしている。)

.. code-block:: shell

  # fufu 直下で実行する
  $ docker-compose run web django-admin.py startproject config .
  Building db
  Step 1/7 : FROM mysql:latest
  latest: Pulling from library/mysql
  177e7ef0df69: Pulling fs layer
  cac25352c4c8: Pull complete
  8585afabb40a: Pull complete
  1e4af4996053: Pull complete
  c326522894da: Pull complete
  9020d6b6b171: Pull complete
  55eb37ec6e5f: Pull complete
  37f3f3d72fbd: Pull complete
  03f098d64268: Pull complete
  46a52a54cfe9: Pull complete
  202bc662895d: Pull complete
  46014f07b258: Pull complete
  Digest: sha256:196c04e1944c5e4ea3ab86ae5f78f697cf18ee43865f25e334a6ffb1dbea81e6
  Status: Downloaded newer image for mysql:latest
   ---> 102816b1ee7d
  Step 2/7 : RUN apt-get clean && apt-get update && apt-get install -y locales locales-all
   ---> Running in 976810f0d320
  Get:1 http://repo.mysql.com/apt/debian stretch InRelease [19.2 kB]
  Get:5 http://repo.mysql.com/apt/debian stretch/mysql-8.0 amd64 Packages [7186 B]
  Ign:2 http://cdn-fastly.deb.debian.org/debian stretch InRelease
  Get:3 http://security-cdn.debian.org/debian-security stretch/updates InRelease [94.3 kB]
  Get:4 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]
  Get:7 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 Packages [464 kB]
  Get:6 http://cdn-fastly.deb.debian.org/debian stretch Release [118 kB]
  Get:8 http://cdn-fastly.deb.debian.org/debian stretch-updates/main amd64 Packages [5152 B]
  Get:9 http://cdn-fastly.deb.debian.org/debian stretch Release.gpg [2434 B]
  Get:10 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 Packages [7089 kB]
  Fetched 7890 kB in 5s (1505 kB/s)
  Reading package lists...
  Reading package lists...
  Building dependency tree...
  Reading state information...
  The following additional packages will be installed:
    libc-l10n
  The following NEW packages will be installed:
    libc-l10n locales locales-all
  0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
  Need to get 7732 kB of archives.
  After this operation, 144 MB of additional disk space will be used.
  Get:1 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libc-l10n all 2.24-11+deb9u3 [820 kB]
  Get:2 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 locales all 2.24-11+deb9u3 [3287 kB]
  Get:3 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 locales-all amd64 2.24-11+deb9u3 [3624 kB]
  debconf: delaying package configuration, since apt-utils is not installed
  Fetched 7732 kB in 3s (2190 kB/s)
  Selecting previously unselected package libc-l10n.
  (Reading database ... 8866 files and directories currently installed.)
  Preparing to unpack .../libc-l10n_2.24-11+deb9u3_all.deb ...
  Unpacking libc-l10n (2.24-11+deb9u3) ...
  Selecting previously unselected package locales.
  Preparing to unpack .../locales_2.24-11+deb9u3_all.deb ...
  Unpacking locales (2.24-11+deb9u3) ...
  Selecting previously unselected package locales-all.
  Preparing to unpack .../locales-all_2.24-11+deb9u3_amd64.deb ...
  Unpacking locales-all (2.24-11+deb9u3) ...
  Setting up libc-l10n (2.24-11+deb9u3) ...
  Setting up locales (2.24-11+deb9u3) ...
  debconf: unable to initialize frontend: Dialog
  debconf: (TERM is not set, so the dialog frontend is not usable.)
  debconf: falling back to frontend: Readline
  Generating locales (this might take a while)...
  Generation complete.
  Setting up locales-all (2.24-11+deb9u3) ...
  Removing intermediate container 976810f0d320
   ---> a38ea7bd8cf5
  Step 3/7 : RUN locale-gen ja_JP.UTF-8
   ---> Running in 45929bff7253
  Generating locales (this might take a while)...
  Generation complete.
  Removing intermediate container 45929bff7253
   ---> 61f34eeb1373
  Step 4/7 : ENV LANG ja_JP.UTF-8
   ---> Running in 0edd321d2039
  Removing intermediate container 0edd321d2039
   ---> 9d081db59b9d
  Step 5/7 : ENV LANGUAGE ja_JP:en
   ---> Running in 3770c59b22a2
  Removing intermediate container 3770c59b22a2
   ---> d1c0b275dfa6
  Step 6/7 : ENV LC_ALL ja_JP.UTF-8
   ---> Running in 751e7ba46d23
  Removing intermediate container 751e7ba46d23
   ---> 8b58a86198fe
  Step 7/7 : RUN ln -sf /usr/share/zoneinfo/Japan /etc/localtime
   ---> Running in 4ee911d35ca8
  Removing intermediate container 4ee911d35ca8
   ---> ae9e35a54368
  Successfully built ae9e35a54368
  Successfully tagged fufu_db:latest
  WARNING: Image for service db was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
  Starting fufu_db ... done
  Building web
  Step 1/9 : FROM python:3
   ---> 7c5fd2af3815
  Step 2/9 : ENV PYTHONUNBUFFERED 1
   ---> Running in 1e7af8a959ba
  Removing intermediate container 1e7af8a959ba
   ---> 145e89eeda68
  Step 3/9 : RUN mkdir /code
   ---> Running in 8d0cf8c1af79
  Removing intermediate container 8d0cf8c1af79
   ---> 50ee8eeb1e91
  Step 4/9 : WORKDIR /code
   ---> Running in 170815b8e495
  Removing intermediate container 170815b8e495
   ---> 2bb29d2d6a2d
  Step 5/9 : ADD requirements.txt /code/
   ---> e7d415a86747
  Step 6/9 : RUN pip install -r requirements.txt
   ---> Running in aca007d771ea
  Collecting Django>=1.11 (from -r requirements.txt (line 1))
    Downloading https://files.pythonhosted.org/packages/36/50/078a42b4e9bedb94efd3e0278c0eb71650ed9672cdc91bd5542953bec17f/Django-2.1.5-py3-none-any.whl (7.3MB)
  Collecting mysqlclient>=1.3.7 (from -r requirements.txt (line 2))
    Downloading https://files.pythonhosted.org/packages/f7/a2/1230ebbb4b91f42ad6b646e59eb8855559817ad5505d81c1ca2b5a216040/mysqlclient-1.3.14.tar.gz (91kB)
  Collecting mysql-connector-python (from -r requirements.txt (line 3))
    Downloading https://files.pythonhosted.org/packages/83/e3/a8782597a548cbaab0e4a24060ecd44da7058a376c0d62182b1aa9797a13/mysql_connector_python-8.0.13-cp37-cp37m-manylinux1_x86_64.whl (8.2MB)
  Collecting pytz (from Django>=1.11->-r requirements.txt (line 1))
    Downloading https://files.pythonhosted.org/packages/61/28/1d3920e4d1d50b19bc5d24398a7cd85cc7b9a75a490570d5a30c57622d34/pytz-2018.9-py2.py3-none-any.whl (510kB)
  Collecting protobuf>=3.0.0 (from mysql-connector-python->-r requirements.txt (line 3))
    Downloading https://files.pythonhosted.org/packages/3a/30/289ead101f94998d88e8961a3548aea29417ae0057be23972483cddebf4f/protobuf-3.6.1-cp37-cp37m-manylinux1_x86_64.whl (1.1MB)
  Requirement already satisfied: setuptools in /usr/local/lib/python3.7/site-packages (from protobuf>=3.0.0->mysql-connector-python->-r requirements.txt (line 3)) (40.6.3)
  Collecting six>=1.9 (from protobuf>=3.0.0->mysql-connector-python->-r requirements.txt (line 3))
    Downloading https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
  Building wheels for collected packages: mysqlclient
    Running setup.py bdist_wheel for mysqlclient: started
    Running setup.py bdist_wheel for mysqlclient: finished with status 'done'
    Stored in directory: /root/.cache/pip/wheels/6d/7d/cb/181963137c414938d4faac9a57c966fb3a6ef675c25641c41a
  Successfully built mysqlclient
  Installing collected packages: pytz, Django, mysqlclient, six, protobuf, mysql-connector-python
  Successfully installed Django-2.1.5 mysql-connector-python-8.0.13 mysqlclient-1.3.14 protobuf-3.6.1 pytz-2018.9 six-1.12.0
  Removing intermediate container aca007d771ea
   ---> 950aa5581bf7
  Step 7/9 : ADD . /code/
   ---> 8d7f1f5cc717
  Step 8/9 : RUN mkdir apps
   ---> c46748e3db9b
  Step 9/9 : WORKDIR apps
   ---> c642446504e2
  Successfully built c642446504e2
  Successfully tagged fufu_web:latest
  WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.


3. 実行後の状態
---------------

.. code-block:: shell

  $ tree fufu
  fufu
  ├── Dockerfile-mysql
  ├── Dockerfile-web
  ├── apps
  │   ├── config
  │   │   ├── __init__.py
  │   │   ├── settings.py
  │   │   ├── urls.py
  │   │   └── wsgi.py
  │   └── manage.py
  ├── docker-compose.yml
  ├── mysql
  │   └── conf.d
  │       └── mysql.cnf
  └── requirements.txt


4. settings に mysql の定義を追記する
-------------------------------------
settings は環境ごとに分けたいので、 ``apps`` 配下に settings ディレクトリを作ってそこ移動する

.. code-block:: bash

  $ tree apps
  apps
  ├── config
  │   ├── __init__.py
  │   ├── urls.py
  │   └── wsgi.py
  ├── manage.py
  └── settings
      └── local.py  # ← ローカル環境用

- settings/local.py

  .. code-block:: python

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'fufu',
            'USER': 'fufu',
            'PASSWORD': 'fufu',
            'HOST': 'db',  # ここは docker-compose ファイルに指定したサービス名でないといけないらしい
            'PORT': 3306,
        }
    }


5. web・db コンテナを起動する
-------------------------------------

.. code-block:: bash

  $ docker-compose up


環境構築メモ
=============
**※1**: default-authentication-plugin=mysql_native_password について
-------------------------------------------------------------------------

- MySQL 8.0.4 からデフォルトの認証 plugin のデフォルト値が ``mysql_native_password`` から ``caching_sha2_password`` へ変更になった
- そのため、 ``default-authentication-plugin`` を指定していない状態で ``caching_sha2_password`` に対応していないクライアント (今回の場合は ``web`` ) から接続しようとすると、
- こんなエラー↓が出て接続できない ( ``docker-compose up`` 時にこうなる)

  .. code-block:: shell

    web_1  | Performing system checks...
    web_1  |
    web_1  | System check identified no issues (0 silenced).
    web_1  | Unhandled exception in thread started by <function check_errors.<locals>.wrapper at 0x7fe293c032f0>
    web_1  | Traceback (most recent call last):
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 216, in ensure_connection
    web_1  |     self.connect()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 194, in connect
    web_1  |     self.connection = self.get_new_connection(conn_params)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/mysql/base.py", line 227, in get_new_connection
    web_1  |     return Database.connect(**conn_params)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/MySQLdb/__init__.py", line 85, in Connect
    web_1  |     return Connection(*args, **kwargs)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/MySQLdb/connections.py", line 208, in __init__
    web_1  |     super(Connection, self).__init__(*args, **kwargs2)
    web_1  | _mysql_exceptions.OperationalError: (2006, "Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory")
    web_1  |
    web_1  | The above exception was the direct cause of the following exception:
    web_1  |
    web_1  | Traceback (most recent call last):
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/utils/autoreload.py", line 225, in wrapper
    web_1  |     fn(*args, **kwargs)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/core/management/commands/runserver.py", line 120, in inner_run
    web_1  |     self.check_migrations()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/core/management/base.py", line 442, in check_migrations
    web_1  |     executor = MigrationExecutor(connections[DEFAULT_DB_ALIAS])
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/migrations/executor.py", line 18, in __init__
    web_1  |     self.loader = MigrationLoader(self.connection)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/migrations/loader.py", line 49, in __init__
    web_1  |     self.build_graph()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/migrations/loader.py", line 212, in build_graph
    web_1  |     self.applied_migrations = recorder.applied_migrations()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/migrations/recorder.py", line 61, in applied_migrations
    web_1  |     if self.has_table():
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/migrations/recorder.py", line 44, in has_table
    web_1  |     return self.Migration._meta.db_table in self.connection.introspection.table_names(self.connection.cursor())
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 255, in cursor
    web_1  |     return self._cursor()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 232, in _cursor
    web_1  |     self.ensure_connection()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 216, in ensure_connection
    web_1  |     self.connect()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/utils.py", line 89, in __exit__
    web_1  |     raise dj_exc_value.with_traceback(traceback) from exc_value
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 216, in ensure_connection
    web_1  |     self.connect()
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/base/base.py", line 194, in connect
    web_1  |     self.connection = self.get_new_connection(conn_params)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/django/db/backends/mysql/base.py", line 227, in get_new_connection
    web_1  |     return Database.connect(**conn_params)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/MySQLdb/__init__.py", line 85, in Connect
    web_1  |     return Connection(*args, **kwargs)
    web_1  |   File "/usr/local/lib/python3.7/site-packages/MySQLdb/connections.py", line 208, in __init__
    web_1  |     super(Connection, self).__init__(*args, **kwargs2)
    web_1  | django.db.utils.OperationalError: (2006, "Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory")


- Django から MySQL に接続する際は、 ``caching_sha2_password`` に対応していない ``mysqlclient`` を使っているようなので、それでも接続できるように、
- ``default-authentication-plugin=mysql_native_password`` の指定が必要 (なんだと思う)
- 後から該当ユーザーの ``default-authentication-plugin`` を変更するにはこう↓

  .. code-block:: shell

    ALTER USER 'fufu'
      IDENTIFIED WITH mysql_native_password
      BY 'fufu';


**※2**: skip-host-cache, skip-name-resolve について
-------------------------------------------------------------

- Docker Hub の MySQL 公式イメージ ``mysql:latest`` からコンテナをそのまま起動すると、
- デフォルト状態で ``/etc/mysql/conf.d/docker.cnf`` の ``[mysqld]`` セクションにこのふたつが定義されている

  - 2018/12/29 の Update で追加されたように (?) 見える

- このふたつがないと、こんな感じ↓で延々とエラーになり、 db コンテナが起動できない

  .. code-block:: shell

    $ docker-compose up
    Creating network "fufu_default" with the default driver
    Creating volume "fufu_db-data" with default driver
    Creating fufu_db ... done
    Creating fufu_web ... done
    Attaching to fufu_db, fufu_web
    db_1   | Initializing database
    db_1   | 2019-01-20T11:55:51.840057Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:51.840160Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.13) initializing of server in progress as process 31

    (中略)

    db_1   | 2019-01-20T11:55:53.441764Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:53.441886Z 0 [ERROR] [MY-013236] [Server] Newly created data directory /var/lib/mysql/ is unusable. You can safely remove it.
    db_1   | 2019-01-20T11:55:53.441926Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:55.045197Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    fufu_db exited with code 1
    db_1   | Initializing database
    db_1   | 2019-01-20T11:55:51.840057Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:51.840160Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.13) initializing of server in progress as process 31
    db_1   | 2019-01-20T11:55:53.441764Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:53.441886Z 0 [ERROR] [MY-013236] [Server] Newly created data directory /var/lib/mysql/ is unusable. You can safely remove it.
    db_1   | 2019-01-20T11:55:53.441926Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:55.045197Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:55:57.680035Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:57.680129Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:55:57.942301Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:55:58.026899Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:55:58.029078Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:55:58.031090Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:55:58.031510Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:55:58.031827Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:55:58.031895Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:55:58.032460Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:55:58.032548Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:55:58.032782Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:55:58.032832Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:58.032975Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:59.851741Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:02.499663Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:02.499753Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:02.758687Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:02.841674Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:02.844213Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:02.846178Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:02.846616Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:02.846911Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:02.846948Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:02.847441Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:02.847506Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:02.847644Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:02.847671Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:02.847769Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:04.258157Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    fufu_db exited with code 1
    db_1   | Initializing database
    db_1   | 2019-01-20T11:55:51.840057Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:51.840160Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.13) initializing of server in progress as process 31
    db_1   | 2019-01-20T11:55:53.441764Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:53.441886Z 0 [ERROR] [MY-013236] [Server] Newly created data directory /var/lib/mysql/ is unusable. You can safely remove it.
    db_1   | 2019-01-20T11:55:53.441926Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:55.045197Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:55:57.680035Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:57.680129Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:55:57.942301Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:55:58.026899Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:55:58.029078Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:55:58.031090Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:55:58.031510Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:55:58.031827Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:55:58.031895Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:55:58.032460Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:55:58.032548Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:55:58.032782Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:55:58.032832Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:58.032975Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:59.851741Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:02.499663Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:02.499753Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:02.758687Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:02.841674Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:02.844213Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:02.846178Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:02.846616Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:02.846911Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:02.846948Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:02.847441Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:02.847506Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:02.847644Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:02.847671Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:02.847769Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:04.258157Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:06.885337Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:06.885434Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:07.153069Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:07.236161Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:07.238540Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:07.240561Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:07.240967Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:07.241151Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:07.241186Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:07.241673Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:07.241732Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:07.241871Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:07.241902Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:07.241995Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:09.048029Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:11.651915Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:11.652007Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:11.912327Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:11.994607Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:11.996767Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:11.998486Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:11.998844Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:11.999016Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:11.999047Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:11.999541Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:11.999601Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:11.999753Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:11.999778Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:11.999874Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:13.815771Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    fufu_db exited with code 1
    db_1   | Initializing database
    db_1   | 2019-01-20T11:55:51.840057Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:51.840160Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.13) initializing of server in progress as process 31
    db_1   | 2019-01-20T11:55:53.441764Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:53.441886Z 0 [ERROR] [MY-013236] [Server] Newly created data directory /var/lib/mysql/ is unusable. You can safely remove it.
    db_1   | 2019-01-20T11:55:53.441926Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:55.045197Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:55:57.680035Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:57.680129Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:55:57.942301Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:55:58.026899Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:55:58.029078Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:55:58.031090Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:55:58.031510Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:55:58.031827Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:55:58.031895Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:55:58.032460Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:55:58.032548Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:55:58.032782Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:55:58.032832Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:58.032975Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:59.851741Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:02.499663Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:02.499753Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:02.758687Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:02.841674Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:02.844213Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:02.846178Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:02.846616Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:02.846911Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:02.846948Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:02.847441Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:02.847506Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:02.847644Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:02.847671Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:02.847769Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:04.258157Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:06.885337Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:06.885434Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:07.153069Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:07.236161Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:07.238540Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:07.240561Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:07.240967Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:07.241151Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:07.241186Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:07.241673Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:07.241732Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:07.241871Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:07.241902Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:07.241995Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:09.048029Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:11.651915Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:11.652007Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:11.912327Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:11.994607Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:11.996767Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:11.998486Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:11.998844Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:11.999016Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:11.999047Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:11.999541Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:11.999601Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:11.999753Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:11.999778Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:11.999874Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:13.815771Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:16.554707Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:16.554806Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:16.812075Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:16.895053Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:16.897313Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:16.899059Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:16.899390Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:16.899657Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:16.899694Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:16.900291Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:16.900354Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:16.900626Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:16.900654Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:16.900826Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:18.720970Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:23.019641Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:23.019736Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:23.282147Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:23.310746Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:23.317461Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:23.319319Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:23.320932Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:23.321207Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:23.321237Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:23.321817Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:23.321890Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:23.322074Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:23.322097Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:23.322247Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:25.183207Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    fufu_db exited with code 1
    db_1   | Initializing database
    db_1   | 2019-01-20T11:55:51.840057Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:51.840160Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.13) initializing of server in progress as process 31
    db_1   | 2019-01-20T11:55:53.441764Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:53.441886Z 0 [ERROR] [MY-013236] [Server] Newly created data directory /var/lib/mysql/ is unusable. You can safely remove it.
    db_1   | 2019-01-20T11:55:53.441926Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:55.045197Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:55:57.680035Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:55:57.680129Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:55:57.942301Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:55:58.026899Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:55:58.029078Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:55:58.031090Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:55:58.031510Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:55:58.031827Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:55:58.031895Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:55:58.032460Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:55:58.032548Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:55:58.032782Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:55:58.032832Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:55:58.032975Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:55:59.851741Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:02.499663Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:02.499753Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:02.758687Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:02.841674Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:02.844213Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:02.846178Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:02.846616Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:02.846911Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:02.846948Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:02.847441Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:02.847506Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:02.847644Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:02.847671Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:02.847769Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:04.258157Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:06.885337Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:06.885434Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:07.153069Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:07.236161Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:07.238540Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:07.240561Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:07.240967Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:07.241151Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:07.241186Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:07.241673Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:07.241732Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:07.241871Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:07.241902Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:07.241995Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:09.048029Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:11.651915Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:11.652007Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:11.912327Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:11.994607Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:11.996767Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:11.998486Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:11.998844Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:11.999016Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:11.999047Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:11.999541Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:11.999601Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:11.999753Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:11.999778Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:11.999874Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:13.815771Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:16.554707Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:16.554806Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:16.812075Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:16.895053Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:16.897313Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:16.899059Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:16.899390Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:16.899657Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:16.899694Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:16.900291Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:16.900354Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:16.900626Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:16.900654Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:16.900826Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:18.720970Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:23.019641Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:23.019736Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:23.282147Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:23.310746Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:23.317461Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:23.319319Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:23.320932Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:23.321207Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:23.321237Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:23.321817Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:23.321890Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:23.322074Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:23.322097Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:23.322247Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:25.183207Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:32.657838Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:32.657925Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:32.919082Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:33.002849Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:33.005166Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:33.006884Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:33.007190Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:33.007365Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:33.007399Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:33.007897Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:33.007959Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:33.008121Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:33.008148Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:33.008248Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:34.819455Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    db_1   | 2019-01-20T11:56:48.777104Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
    db_1   | 2019-01-20T11:56:48.777198Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
    db_1   | mysqld: Table 'mysql.plugin' doesn't exist
    db_1   | 2019-01-20T11:56:49.036122Z 0 [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
    db_1   | 2019-01-20T11:56:49.117775Z 0 [Warning] [MY-010015] [Repl] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
    db_1   | 2019-01-20T11:56:49.120188Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
    db_1   | 2019-01-20T11:56:49.122183Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
    db_1   | 2019-01-20T11:56:49.122556Z 0 [Warning] [MY-010441] [Server] Failed to open optimizer cost constant tables
    db_1   | 2019-01-20T11:56:49.122852Z 0 [ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist
    db_1   | 2019-01-20T11:56:49.122890Z 0 [Warning] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-003543 - The mysql.component table is missing or has an incorrect definition.
    db_1   | 2019-01-20T11:56:49.123460Z 0 [ERROR] [MY-010326] [Server] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
    db_1   | 2019-01-20T11:56:49.123544Z 0 [Warning] [MY-010952] [Server] The privilege system failed to initialize correctly. If you have upgraded your server, make sure you're executing mysql_upgrade to correct the issue.
    db_1   | 2019-01-20T11:56:49.123830Z 0 [Warning] [MY-010357] [Server] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
    db_1   | 2019-01-20T11:56:49.123863Z 0 [ERROR] [MY-010361] [Server] Fatal error: Illegal or unknown default time zone 'Asia/Tokyo'
    db_1   | 2019-01-20T11:56:49.124062Z 0 [ERROR] [MY-010119] [Server] Aborting
    db_1   | 2019-01-20T11:56:50.944319Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.13)  MySQL Community Server - GPL.
    fufu_db exited with code 1


- わたしの場合は、 docker-compose ファイルでこう↓しているので、デフォルト状態では存在した ``/etc/mysql/conf.d/docker.cnf`` を抹殺してしまっている

  .. code-block:: yaml

    volumes:
      - "./mysql/conf.d:/etc/mysql/conf.d"

- しかたがないので、自分の ``mysql.cnf`` に転記することにした
- このふたつがないとどうしてこのエラーになるのかわたしにはわかりません、だって全然関係ないこと言ってるように見えるのに...
- このふたつの説明はここです: https://dev.mysql.com/doc/refman/8.0/en/host-cache.html


使い方メモ
===========

再び image ビルドしたくなったら
-------------------------------

.. code-block:: shell

  $ docker-compose run web django-admin.py startproject config .

とか

.. code-block:: shell

  $ docker-compose run web django-admin.py startproject config .
  $ docker-compose up

したあとに、再び image ビルドしたくなったら、

.. code-block:: shell

  # web をビルド
  $ docker-compose build web

  # db をビルド
  $ docker-compose build db
