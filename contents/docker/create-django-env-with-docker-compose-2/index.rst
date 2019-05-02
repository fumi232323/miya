.. article::
   :title: Docker Compose で Django 環境をつくろう その2
   :date: 2019-05-01
   :category: docker
   :tags:
   :canonical_url: /docker/create-django-env-with-docker-compose-2/
   :draft: false


=============================================
Docker Compose で Django 環境をつくろう その2
=============================================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス・参考サイト書籍
=============================
ありがとうございました


- Python プロフェッショナルプログラミング第３版 > 01-01 Python のセットアップ

- `ubuntu Docker Official Images <https://hub.docker.com/_/ubuntu>`_

  - ``en_US.utf8`` を ``apt-get`` するくだりを参考にした

    - Ubuntu イメージにはデフォルトで ``C``, ``C.UTF-8``, and ``POSIX`` locales が含まれている、これで UTF-8 locale を使うほとんどのひとには十分なはずだけれども、
    - 足りない場合は、``locales`` package からインストールしてね

- `Best practices for writing Dockerfiles <https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>`_

  - ``RUN`` で ``apt-get`` するときはこうすると良さそう

    - ``apt-get upgrade`` , ``dist-upgrade`` するのは避けよう
    - ``apt-get update && apt-get install -y`` はセットでやろう

      - こうすると、 latest package versions が install できる
      - このテクニックは、 ``cache busting`` として知られているらしい

    - packages はアルファベット順に並べる

      - 重複に気づけるし、後から足したり減らしたりするのが容易になるから

    - 最後に ``rm -rf /var/lib/apt/lists/*`` して apt キャッシュを clean up すると、 image size を減らせる

  - 良い見本

    .. code-block:: docker

      RUN apt-get update && apt-get install -y \
          aufs-tools \
          automake \
          build-essential \
          curl \
          dpkg-sig \
          libcap-dev \
          libsqlite3-dev \
          mercurial \
          reprepro \
          ruby1.9.1 \
          ruby1.9.1-dev \
          s3cmd=1.1.* \
       && rm -rf /var/lib/apt/lists/*


- `Docker development best practices <https://docs.docker.com/develop/dev-best-practices/>`_

  - ``RUN`` で ``apt-get`` するときはこうすると良さそう

    - 2行で書くより、1行で書く
    - 1行で書くと、イメージを小さくできる ( ``layers`` がなんのことなのかはよくわからない)
    - `The first creates two layers in the image, while the second only creates one.`

      .. code-block:: docker

        RUN apt-get -y update
        RUN apt-get install -y python


      .. code-block:: docker

        RUN apt-get -y update && apt-get install -y python


- `くろのて勉強会 > DRF勉強会 (全6回) > djample <https://github.com/righ/djample/>`_
- 現場で使える Django の教科書 > D: Docker でラクラク開発
- `Docker Ubuntu18.04でtzdataをinstallするときにtimezoneの選択をしないでinstallする <https://qiita.com/yagince/items/deba267f789604643bab>`_

  - 途中で何も尋ねてほしくないときは、 ``ENV DEBIAN_FRONTEND=noninteractive`` すると良さそう

- `ModuleNotFoundError: No module named '_ctypes' の解決方法 <https://stackoverflow.com/questions/27022373/python3-importerror-no-module-named-ctypes-when-using-value-from-module-mul/48045929>`_

  - 事前に ``libffi-dev`` パッケージのインストールが必要

- `[Linux]タイムゾーン(Timezone)をUTCから日本標準時(JST)に変更する <https://www.t3a.jp/blog/infrastructure/set-timezone/>`_

  - シンボリックリンクを張り替える
  - ``ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime``

- `2.2. Python のビルド <https://docs.python.org/ja/3/using/unix.html#building-python>`_

  - ``make install`` の代わりに ``make altinstall`` 推奨
  - `警告 make install は python3 バイナリを上書きまたはリンクを破壊してしまうかもしれません。そのため、make install の代わりに exec_prefix/bin/pythonversion のみインストールする make altinstall が推奨されています。` だそうです。


つくりかた
==========

実際には、 PostgreSQL か MySQL かどちらか使うほうのみを生きとする。

構成
-----

.. code-block:: bash

  $ tree fufufu
  fufufu
  ├── Dockerfile-app        # 1. Django プロジェクト作るよう Dockerfile
  ├── Dockerfile-mysql      # 2. MySQL よう Dockerfile
  ├── Dockerfile-postgres   # 3. PostgreSQL よう Dockerfile
  ├── docker-compose.yml    # 4. Composeファイル
  └── requirements.txt      # 5. requirements.txt


設定ファイル
-------------

1. ``Dockerfile-app`` : Django プロジェクトを入れるコンテナ

    .. code-block:: docker

      FROM ubuntu:18.04

      # インストール中に何も尋ねてくるな
      ENV DEBIAN_FRONTEND=noninteractive

      # Python の環境変数を設定
      # stdout, stderr のバッファを無効に
      ENV PYTHONUNBUFFERED 1
      # ソースモジュールのインポート時に .pyc ファイルを作成しない
      ENV PYTHONDONTWRITEBYTECODE 1

      # locales をインストールして設定する
      RUN apt-get clean && apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
          && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
      ENV LANG en_US.UTF-8
      ENV LANGUAGE en_US:en
      ENV LC_ALL en_US.UTF-8

      # タイムゾーンに JST を設定
      RUN ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

      # Python ビルドに必要な deb パッケージのインストール
      # `libffi-dev`: 3.6 では不要 (?) 、3.7 では必要
      RUN apt-get clean && apt-get update && apt-get install -y \
          build-essential \
          python3-dev \
          libsqlite3-dev \
          libreadline6-dev \
          libgdbm-dev \
          zlib1g-dev \
          libbz2-dev \
          sqlite3 \
          tk-dev \
          zip \
          libssl-dev \
          libffi-dev \
          wget \
       && rm -rf /var/lib/apt/lists/*

      # Python をソースファイルからビルドしてインストール
      # `make altinstall`: `make install` の代わりに推奨
      RUN wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz \
          && tar xf Python-3.7.3.tgz \
          && cd Python-3.7.3 \
          && ./configure --prefix=/opt/python3.7.3 \
          && make \
          && make altinstall

      # Python のシンボリックリンクを作成
      RUN ln -s /opt/python3.7.3/bin/python3.7 /usr/local/bin/python

      # pip のシンボリックリンクを作成
      RUN ln -s /opt/python3.7.3/bin/pip3.7 /usr/local/bin/pip

      # pip をアップグレード
      RUN pip install -U pip

      # mysqlclient のインストールに必要なので、インストールしておく
      RUN apt-get clean && apt-get update && apt-get install -y \
          default-libmysqlclient-dev \
       && rm -rf /var/lib/apt/lists/*


2. ``Dockerfile-mysql`` : MySQL を入れるコンテナ

    .. code-block:: docker

      FROM mysql:latest

      # locales をインストールして設定する
      RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
          && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
      ENV LANG en_US.UTF-8
      ENV LANGUAGE en_US:en
      ENV LC_ALL en_US.UTF-8

      # タイムゾーンに JST を設定
      RUN ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime


3. ``Dockerfile-postgres`` : PostgreSQL を入れるコンテナ

    .. code-block:: docker

      FROM postgres:latest

      ## locales をインストールして設定する (PostgreSQL の場合は、公式イメージで実行済み)
      ## https://github.com/docker-library/postgres/blob/85aadc08c347cd20f199902c4b8b4f736341c3b8/11/Dockerfile
      #RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
      #  && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
      #ENV LANG en_US.UTF-8
      ENV LANGUAGE en_US:en
      ENV LC_ALL en_US.UTF-8

      # タイムゾーンに JST を設定
      RUN ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime


4. ``docker-compose.yml`` : Composeファイル

    .. code-block:: yaml

      version: '3'

      services:
        app:
          container_name: fufufu_app
          build:
            context: .
            dockerfile: Dockerfile-app
          volumes:
            - .:/fufufu
          working_dir: /fufufu
          command: bash -c "pip install -r requirements.txt && bash"
          ports:
            - "8000:8000"
          tty: true  # 起動し続ける
          depends_on:
            - mysql
            - postgres

        mysql:
          container_name: fufufu_mysql
          build:
            context: .
            dockerfile: Dockerfile-mysql
          command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
          restart: always
          volumes:
            - "mysql-data:/var/lib/mysql"
          environment:
            MYSQL_ROOT_PASSWORD: fufufu
            MYSQL_DATABASE: fufufu
            MYSQL_USER: fufufu
            MYSQL_PASSWORD: fufufu

        postgres:
          container_name: fufufu_postgres
          build:
            context: .
            dockerfile: Dockerfile-postgres
          restart: always
          volumes:
            - "postgres-data:/var/lib/postgresql/data"
          environment:
            POSTGRES_DB: fufufu
            POSTGRES_USER: fufufu
            POSTGRES_PASSWORD: fufufu

      volumes:
        mysql-data:
        postgres-data:


5. ``requirements.txt`` : requirements.txt


    .. code-block:: yaml

      Django>=2.1
      mysqlclient  # MySQL のドライバー
      psycopg2-binary  # PostgreSQL のドライバー


起動
----

.. code-block:: bash

  $ docker-compose up
  # 再度 image ビルドからやり直したい
  $ docker-compose up --build


つかいかた
==========

app コンテナ
------------

.. code-block:: bash

  # Django プロジェクトのコンテナに入る
  $ docker container exec -it fufufu_app bash
  # Django プロジェクトを作成する
  $ cd /fufufu
  $ /opt/python3.7.3/lib/python3.7/site-packages/django/bin/django-admin.py startproject config .
  # runserver する ( settings.py の ``ALLOWED_HOSTS`` に ``'0.0.0.0'`` を書いておかないと ``DisallowedHost`` になります)
  $ python manage.py runserver 0.0.0.0:8000


MySQL, PostgreSQL コンテナ
--------------------------

.. code-block:: bash

  # MySQL のコンテナに入る
  $ docker container exec -it fufufu_mysql bash
  # MySQL に入る
  $ mysql -u fufufu -D fufufu -p

  # PostgreSQL のコンテナに入る
  $ docker container exec -it fufufu_postgres bash
  # PostgreSQL に入る
  $ psql -U fufufu fufufu


感想
====
- ``Python をソースファイルからビルドしてインストール`` するのは時間がかかる
- Django プロジェクトを作成するのに、venv を作らず、

  - ``/opt/python3.7.3/lib/python3.7/site-packages/django/bin/django-admin.py startproject config .`` じゃなくて、 ``django-admin.py startproject config .`` できる方法ないのだろうか...
  - 手動で ``/opt/python3.7.3/lib/python3.7/site-packages/django/bin/`` に PATH を通せばできるけどそうじゃなくて自動でやってくれないのかな...


その他
======

Python 公式イメージのルーツ
-----------------------------

python:latest, python:3 => buildpack-deps:stretch => debian:stretch

- ubuntu は Debian-based ということだけれど、 Debian と ubuntu はどんな風に違うんだろうなあ
- MySQL, PostgreSQL の公式イメージも Debian (debian:stretch-slim) だった

Docker Hub
^^^^^^^^^^

- `python Docker Official Images <https://hub.docker.com/_/python>`_
- `buildpack-deps Docker Official Images <https://hub.docker.com/_/buildpack-deps>`_

  - `A collection of common build dependencies used for installing various modules, e.g., gems.`

- `Debian Docker Official Images <https://hub.docker.com/_/debian>`_