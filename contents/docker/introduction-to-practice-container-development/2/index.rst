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


2.3 Docker コンテナの操作
=========================
Docker コンテナは外から見ると仮想環境、ファイルシステムとアプリケーションが同梱されている箱のようなもの。

2.3.1 Docker コンテナのライフサイクル
-------------------------------------
Docker コンテナは、以下の3つの状態のいずれかに分類される。

.. list-table::
  :widths: auto
  :stub-columns: 1

  * - 実行中
    - ``$ docker container run`` で起動した状態。
  * - 停止
    - 停止しても、ディスクにコンテナ終了時の状態は保持される。停止したコンテナは再実行可能。
  * - 破棄
    - - 停止したコンテナは明示的に破棄しない限りディスクに残り続ける。どんどんたまる。
      - 完全に不要なコンテナは破棄するほうが望ましい。
      - 一度破棄したコンテナを再び開始することはできない。


2.3.2 docker container run --- コンテナの作成と実行
----------------------------------------------------
Docker イメージからコンテナを作成、実行するコマンド。

.. code-block:: console

  $ docker container run [options] イメージ名[:タグ名] [コマンド] [コマンド引数...]

.. code-block:: console

  $ docker container run [options] イメージID [コマンド] [コマンド引数...]

.. note::

  コンテナをバックグラウンドで実行 → HTTP リクエストしてみる → 停める

  .. code-block:: console

    $ docker container run -d -p 9001:8080 example/echo:latest
    $ curl http://localhost:9001/
    $ docker container stop $(docker container ls --filter "ancestor=example/echo" -q)


docker container run 時に引数を与える
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

  $ docker image pull alpine:3.7
  # docker container run -it alpine:3.7  # シェルに入る
  $ docker container run -it alpine:3.7 uname -a

名前付きコンテナ
^^^^^^^^^^^^^^^^

``NAMES`` は適当な単語で作られた名前が自動でつけられる。

.. code-block:: console

  $ docker container ls
  CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
  77699bc8d7cd        example/echo:latest   "go run /echo/main.go"   4 seconds ago       Up 2 seconds        0.0.0.0:9001->8080/tcp   modest_saha

コンテナに好きな名前をつけられる。

.. code-block:: console

  $ docker container run --name [好きなコンテナ名] [イメージ名]:[タグ]

.. code-block:: console

  $ docker container run -t -d --name gihyo-echo example/echo:latest
  4864fcaf10802340449f50364891cc48b99e90538f04d8e601c5c0397ff11917
  $ docker container ls
  CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
  4864fcaf1080        example/echo:latest   "go run /echo/main.go"   3 seconds ago       Up 2 seconds                                 gihyo-echo
  77699bc8d7cd        example/echo:latest   "go run /echo/main.go"   4 minutes ago       Up 4 minutes        0.0.0.0:9001->8080/tcp   modest_saha

- 開発時は便利だが、本番環境ではあまり使わない
- 同名のコンテナを新たに実行するには既存の同名コンテナを削除する必要があるため


コマンド実行時の頻出オプション
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:-i: docker 起動後にコンテナ側の標準入力をつなぎっぱなしにする。シェルに入ってコマンド実行ができる。
:-t: 擬似端末を有効にする。
:-it: -i と -t はセットで使うことが多い。
:--rm: コンテナ終了時にコンテナを破棄する。
:-v: ホストとコンテナ間でディレクトリ、ファイルを共有する


2.3.3 docker container ls --- コンテナの一覧
--------------------------------------------
実行中及び終了したコンテナの一覧を表示するコマンド

.. code-block:: console

  $ docker container ls [options]

オプションなしで実行すると、実行中のコンテナ一覧が表示される

.. code-block:: console

  $ docker container run -t -d -p 8080 --name fumi23 example/echo:latest
  81e3a724ae7c730eea14b86edf354c9aad4bced96e272d1fce238760080a23b6
  $ docker container run -t -d -p 8080 --name fumi45 example/echo:latest
  db029554bc5fc2e23a724892bef867c613ae5dba861e50de48914bcde23ebaf1
  $ docker container ls
  CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
  db029554bc5f        example/echo:latest   "go run /echo/main.go"   21 seconds ago      Up 21 seconds       0.0.0.0:32769->8080/tcp   fumi45
  81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   44 seconds ago      Up 43 seconds       0.0.0.0:32768->8080/tcp   fumi23


.. list-table:: 一覧の表示項目
  :widths: auto
  :header-rows: 1

  * - 項目
    - 内容
  * - CONTAINER ID
    - コンテナに付与される一意の ID
  * - IMAGE
    - コンテナ作成に使用された Docker イメージ
  * - COMMAND
    - コンテナで実行されているアプリケーションのプロセス
  * - CREATED
    - コンテナが作成されてから経過した時間
  * - STATUS
    - Up (実行中), Exited(終了) といったコンテナの実行状態
  * - PORTS
    - ホストのポートとコンテナポートの紐づけ (ポートフォワーディング)
  * - NAMES
    - コンテナにつけられた名前

コンテナIDだけを抽出する
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

  $ docker container ls -q
  db029554bc5f
  81e3a724ae7c
