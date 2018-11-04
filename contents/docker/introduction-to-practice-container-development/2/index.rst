.. article::
   :date: 2018-11-04
   :title: Docker/Kubernetes 実践コンテナ開発入門 - 2. Docker コンテナのデプロイ
   :category: docker
   :tags:
   :canonical_url: /docker/introduction-to-practice-container-development/2/
   :draft: true


============================
2. Docker コンテナのデプロイ
============================


2.1 コンテナでアプリケーションを実行する
========================================

.. list-table:: Docker イメージと Docker コンテナの関係性
  :widths: auto
  :header-rows: 1

  * - 名称
    - 役割
  * - Docker イメージ
    - - Docker コンテナを構成するファイルシステムや、実行するアプリケーションや設定をまとめたもの
      - コンテナを作成するために利用されるテンプレートとなるもの
  * - Docker コンテナ
    - - Docker イメージを基に作成され、具現化されたファイルシステムとアプリケーションが実行されている状態
      - ひとつの Docker イメージから複数のコンテナを生成できる

- まずは、イメージから作成する


2.1.1 Docker イメージと Docker コンテナの基本
---------------------------------------------

基本の流れ
^^^^^^^^^^

1. Docker イメージをダウンロード

  .. code-block:: console

    FumienoMacBook-Pro:docker-work fumi23$ docker image pull gihyodocker/echo:latest
    latest: Pulling from gihyodocker/echo
    723254a2c089: Pull complete
    abe15a44e12f: Pull complete
    409a28e3cc3d: Pull complete
    503166935590: Pull complete
    abe52c89597f: Pull complete
    ce145c5cf4da: Pull complete
    96e333289084: Pull complete
    39cd5f38ffb8: Pull complete
    22860d04f4f1: Pull complete
    7528760e0a03: Pull complete
    Digest: sha256:4520b6a66d2659dea2f8be5245eafd5434c954485c6c1ac882c56927fe4cec84
    Status: Downloaded newer image for gihyodocker/echo:latest


2. Docker イメージを実行

  .. code-block:: console

    FumienoMacBook-Pro:docker-work fumi23$ docker container run -t -p 9000:8080 gihyodocker/echo:latest
    2018/10/01 13:53:03 start server

  - ポートフォワーディング設定をしている

    - Docker 実行環境の 9000 ポート経由で HTTP リクエストを受けられるようになっている

3. 別のターミナルからアクセスしてみる

  .. code-block:: console

    FumienoMacBook-Pro:~ fumi23$ curl http://localhost:9000
    Hello Docker!!


  .. code-block:: console

    FumienoMacBook-Pro:docker-work fumi23$ docker container run -t -p 9000:8080 gihyodocker/echo:latest
    2018/10/01 13:53:03 start server
    2018/10/01 13:56:44 received request


4. 停止する

  .. code-block:: console

    $ docker stop $(docker container ls -q)


2.1.2 簡単なアプリケーションと Docker イメージをつくる
-------------------------------------------------------

Docker コンテナがどのように作られ、実行されているかのイメージをつかむ。
Go 言語で簡単な Web サーバーを書き、 Docker コンテナ嬢で動作させてみましょう。

用意するもの
^^^^^^^^^^^^^
- main.go
- Dockerfile

Dockerfileの説明
^^^^^^^^^^^^^^^^^
- Dockerfile には、 Docker 独自の DSL (ドメイン固有言語) を使ってイメージの構成を定義する。
- ``FROM`` や ``RUN`` といったキーワードは「インストラクション (命令) 」と呼ばれている。

  .. list-table:: Dockerfile のインストラクション
    :widths: auto
    :header-rows: 1

    * - インストラクション名
      - 説明
    * - FROM
      - - 作成する Docker イメージのベースとなるイメージを指定する。
        - Dockerfile でイメージをビルドする際、まず最初に ``FROM`` で指定されたイメージをダウンロードしてから実行される。
        - Docker は、デフォルトで ``FROM`` の取得先として ``Docker Hub`` のレジストリを参照する。
    * - RUN
      - - Docker イメージビルド時に、 Docker コンテナで実行するコマンドを定義します。
        - ``RUN`` の引数には Docker コンテナ内で実行するコマンドをそのまま指定する。
    * - COPY
      - Docker を動作させているホストマシン上のファイルやディレクトリを Docker コンテナ内にコピーするためのインストラクション。
    * - CMD
      - - Docker コンテナとして実行する際に、コンテナ内で実行するプロセスを指定する。
        - ``CMD`` はコンテナ起動時に１度実行される。
        - ``CMD`` で指定した命令は、 docker container run の指定で実行時に上書きできる。
    * - LABEL
      - イメージの作者名記入などに使う。
    * - ENV
      - Dockerfile をもとに生成した Docker コンテナ内で使える環境変数を指定する。
    * - ARG
      - ビルド時に情報を埋め込むために使う。イメージビルドのときだけ使用できる一時的な環境変数。


Docker イメージをビルドする
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Docker イメージを作成するためのコマンド

.. code-block:: console

  $ docker image build -t 名前空間/イメージ名[:タグ名] Dockerfile配置ディレクトリのパス


実行すると、

.. code-block:: console

  $ docker image build -t example/echo:latest .
  Sending build context to Docker daemon  3.072kB
  Step 1/4 : FROM golang:1.9
  1.9: Pulling from library/golang
  55cbf04beb70: Pull complete
  1607093a898c: Pull complete
  9a8ea045c926: Pull complete
  d4eee24d4dac: Pull complete
  9c35c9787a2f: Pull complete
  8b376bbb244f: Pull complete
  0d4eafcc732a: Pull complete
  186b06a99029: Pull complete
  Digest: sha256:8b5968585131604a92af02f5690713efadf029cc8dad53f79280b87a80eb1354
  Status: Downloaded newer image for golang:1.9
   ---> ef89ef5c42a9
  Step 2/4 : RUN mkdir /echo
   ---> Running in 4da08e7c5693
  Removing intermediate container 4da08e7c5693
   ---> 7caf124fb4d3
  Step 3/4 : COPY main.go /echo
   ---> 73db87b05d43
  Step 4/4 : CMD ["go", "run", "/echo/main.go"]
   ---> Running in 3db24ec2a7c7
  Removing intermediate container 3db24ec2a7c7
   ---> 294c33d2b845
  Successfully built 294c33d2b845
  Successfully tagged example/echo:latest
  $ docker image ls
  REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
  example/echo                     latest              294c33d2b845        37 seconds ago      750MB
  golang                           1.9                 ef89ef5c42a9        3 months ago        750MB


- ``ENTRYPOINT`` というものを使うと、コマンド実行が便利になるらしい


2.1.3 Docker コンテナを実行する
--------------------------------

- コンテナを実行する

  .. code-block:: console

    $ docker container run example/echo:latest
    2018/11/04 10:05:45 start server


  - 終了は、 ``Ctrl + C`` (やってみたけど終わらないな...)

- バックグランドでコンテナを実行させる

  .. code-block:: console

    $ docker container run -d example/echo:latest
    449ccdc8c99e72ecd791b036417632ec3e7944f1e7ab14c5b96d7e4caec0e58b

  - ハッシュ値のような文字列は、 Docker コンテナのID
  - コンテナのID は、コンテナ実行時に付与される一意な ID

- 停止する

  .. code-block:: console

    $ docker container stop $(docker container ls --filter "ancestor=example/echo" -q)
    449ccdc8c99e


- 現在実行中のコンテナの一覧を表示する

  .. code-block:: console

    $ docker container ls
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS               NAMES
    449ccdc8c99e        example/echo:latest   "go run /echo/main.go"   2 minutes ago       Up 2 minutes                            determined_zhukovsky


ポートフォワーディング
^^^^^^^^^^^^^^^^^^^^^^
ホストマシンのポートをコンテナポートに紐づける。
コンテナの外から来た通信をコンテナポートに転送することができる。

- ホスト側の 9000 番ポートをコンテナ側の 8080 番ポートにポートフォワーディングする。

  .. code-block:: console

    $ docker container run -d -p 9000:8080 example/echo:latest
    b113261a42b8fb110cd1984904dccfe859067abd078637ff37804ad5f00c3ff5

  - ``-p {ホスト側ポート}:{コンテナポート}``
  - ホスト側のポートは省略できる。省略すると空いているポートが自動的に割り当てられる。

- ホスト側のポートに curl で GET リクエストしてみる

  .. code-block:: console

    $ curl http://localhost:9000/
    Hello Docker!!


2.2 Docker イメージの操作
=========================

:Docker イメージ: Docker コンテナを作成するためのテンプレート
:Dockerfile: イメージを構築するための手順を記述したファイル
:Docker イメージをビルドする: イメージを構築する

- Docker のヘルプを表示する

  .. code-block:: console

    $ docker help

- Docker のイメージ操作に関するコマンドのヘルプを表示する

  .. code-block:: console

    $ docker image --help


2.2.1 docker image build --- イメージのビルド
---------------------------------------------

- Dockerfile をもとに Docker イメージを作成する

  .. code-block:: console

    $ docker image build -t イメージ名[:タグ名] Dockerfile配置ディレクトリのパス

  - ``-t イメージ名[:タグ名]`` : Docker を利用する上でほぼ必須。

- Dockerfile という名前ではない Dockerfile を指定して Docker イメージを作成する

  .. code-block:: console

    $ docker image build -f {Dockerfile名} -t イメージ名[:タグ名] Dockerfile配置ディレクトリのパス

- イメージをビルド時に、 ``FROM`` で指定したベースイメージを強制的に再取得させる。

  .. code-block:: console

    $ docker image build --pull=true -t example/echo:latest

  - 実際の運用では、 ``latest`` ではなく、タグ付けされたイメージを利用することがほとんど

2.2.2 docker search --- イメージの検索
--------------------------------------

Docker Hub
^^^^^^^^^^

- Docker イメージのレジストリ
- ユーザーや組織が GitHub と同様にリポジトリを持つことができる
- リポジトリでそれぞれの Docker イメージを管理していく
- 全てのイメージのベースとなるような OS (CentOS や Ubuntu) のリポジトリ、言語のランタイムや著名なミドルウェアのイメージのリポジトリなどたくさんある
- 全ての Docker イメージを自前で用意する必要はない、ほかの人が作ったものを活用していく

検索する
^^^^^^^^^^

.. code-block:: console

  $ docker search [options] 検索キーワード

- mysql を検索する

  .. code-block:: console

    $ docker search --limit 5 mysql
    NAME                         DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
    mysql                        MySQL is a widely used, open-source relation…   7249                [OK]
    mysql/mysql-server           Optimized MySQL Server Docker images. Create…   535                                     [OK]
    zabbix/zabbix-server-mysql   Zabbix Server with MySQL database support       136                                     [OK]
    mysql/mysql-cluster          Experimental MySQL Cluster Docker images. Cr…   33
    circleci/mysql               MySQL is a widely used, open-source relation…   7


  - スターの降順で表示される
  - ``--limit 5`` : 表示件数を5件に制限する
  - 名前空間はオーナー名
  - 公式リポジトリは、名前空間が表示されない
  - 公式リポジトリの名前空間には一律で ``library`` がついているので、正式名称は ``library/mysql``

- リリースされているタグの一覧を表示する

  .. code-block:: console

    $ curl -s 'https://hub.docker.com/v2/repositories/library/golang/tags/?page_size=10' | jq -r '.results[].name'
    1.10
    1.10.5
    latest
    1
    1.11
    1.11.2
    1.10-alpine3.7
    1.10.5-alpine3.7
    1.10-alpine
    1.10.5-alpine


2.2.3 docker image pull --- イメージの取得
------------------------------------------

Docker レジストリから Docker イメージをダウンロードする

.. code-block:: console

  $ docker image pull [options] リポジトリ名[:タグ名]

- 指定するリポジトリ名とタグ名は Docker Hub に存在するものを指定する
- jenkins の Docker イメージをダウンロードする

  .. code-block:: console

    $ docker image pull jenkins:latest


  - タグ名を省略した場合は、デフォルトタグ (多くは latest) が利用される
  - ダウンロードしてきたイメージは、そのまま Docker コンテナとして利用できる


2.2.4 docker image ls --- イメージの一覧
----------------------------------------

Docker ホストに保持されているイメージの一覧を表示する

.. code-block:: console

  $ docker image ls [options] [リポジトリ名[:タグ名]]

- Docker ホスト: Docker デーモンを実行しているホスト環境のこと
- リモートから pull してきたイメージも、自分でビルドしたイメージも両方表示される

  .. code-block:: console

    $ docker image ls
    REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
    example/echo                     latest              ed899b24590f        3 hours ago         750MB
    jenkins                          latest              cd14cecfdb3a        3 months ago        696MB
    golang                           1.9                 ef89ef5c42a9        3 months ago        750MB
    gihyodocker/echo                 latest              3dbbae6eb30d        10 months ago       733MB

  - ``IMAGE ID`` : イメージのID。コンテナのIDとは違うものなので、混同しないこと。


2.2.5 docker image tag --- イメージのタグ付け
---------------------------------------------

Docker イメージのバージョン
^^^^^^^^^^^^^^^^^^^^^^^^^^^
イメージのバージョンとは、正確にはイメージIDのこと

- イメージのビルドの度に、別の ``IMAGE ID`` が割り振られる。

  .. code-block:: console

    $ docker image ls
    REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
    example/echo                     latest              ed899b24590f        3 hours ago         750MB
    <none>                           <none>              294c33d2b845        3 hours ago         750MB

  - ひとつのタグに紐づけられるイメージはひとつまで (上の例だと ``latest`` )
  - 古いイメージはタグとの紐づけが解除されて ``<none>`` になる

イメージIDへのタグ付け
^^^^^^^^^^^^^^^^^^^^^^
イメージID にタグ名という形で別名をつけることができる

.. code-block:: console

  $ docker image tag 元イメージ名[:タグ] 新イメージ名[:タグ]

- ある特定のイメージIDを持つ Docker イメージを識別しやすくするために使う。
- ``latest`` は Git で言うところの master ブランチのようなもの。常に最新のイメージ。
- ``example/echo`` の ``latest`` に 0.1.0 のタグをつける

  .. code-block:: console

    $ docker image tag example/echo:latest example/echo:0.1.0
    REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
    example/echo                     0.1.0               ed899b24590f        3 hours ago         750MB
    example/echo                     latest              ed899b24590f        3 hours ago         750MB
    <none>                           <none>              294c33d2b845        3 hours ago         750MB


2.2.6 docker image push --- イメージの公開
-------------------------------------------
Docker イメージを Docker Hub などのレジストリに登録する

.. code-block:: console

  $ docker image push [options] リポジトリ名[:タグ]


Docker Hub にイメージを push する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Docker Hub にログインする

    .. code-block:: console

      $ docker login -u your_docker_id -p your_docker_pw

2. 名前空間を自分のリポジトリ名にする

    .. code-block:: console

      $ docker image tag example/echo:latest fumi23/echo:latest
      $ docker image ls
      REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
      example/echo                     0.1.0               ed899b24590f        4 hours ago         750MB
      example/echo                     latest              ed899b24590f        4 hours ago         750MB
      fumi23/echo                      latest              ed899b24590f        4 hours ago         750MB

    - Docker Hub は、自分が所有している、または、所属している organization のリポジトリにしか push できない

3. Docker Hub に push する

    .. code-block:: console

      $ docker image push fumi23/echo:latest
      The push refers to repository [docker.io/fumi23/echo]
      b2aff6d696c0: Preparing
      f18abb5d7b45: Preparing
      f18abb5d7b45: Pushed
      latest: digest: sha256:834be6348517746b53f3d44c56b580a0cea74161b86426cc006b1c066c48e047 size: 2417
