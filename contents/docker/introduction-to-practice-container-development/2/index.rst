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

  .. code-block:: go

    package main

    import (
        "fmt"
        "log"
        "net/http"
    )

    func main() {
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            log.Println("received request")
            fmt.Fprintf(w, "Hello Docker!!")
        })

        log.Println("start server")
        server := &http.Server{Addr: ":8080"}
        if err := server.ListenAndServe(); err != nil {
            log.Println(err)
        }
    }


- Dockerfile

  .. code-block:: python

    FROM golang:1.9

    RUN mkdir /echo
    COPY main.go /echo

    CMD ["go", "run", "/echo/main.go"]


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

- ``-d``: バックグランドでコンテナを実行させる

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


filter を使う
^^^^^^^^^^^^^
特定の条件に一致するものだけを抽出する

.. code-block:: console

  $ docker container ls --filter "filter名=値"

- コンテナ名で抽出する

  .. code-block:: console

    $ docker container ls --filter "name=fumi45"
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    db029554bc5f        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32769->8080/tcp   fumi45

- イメージ名で抽出する

  .. code-block:: console

    $ docker container ls --filter "ancestor=example/echo"
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    db029554bc5f        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32769->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23


終了したコンテナを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^
終了したコンテナも含めたコンテナの一覧を取得する

  .. code-block:: console

    $ docker container ls -a
    CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                    PORTS                     NAMES
    db029554bc5f        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 days                 0.0.0.0:32769->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 days                 0.0.0.0:32768->8080/tcp   fumi23
    4864fcaf1080        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               gihyo-echo
    77699bc8d7cd        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               modest_saha
    ...


2.3.4 docker container stop --- コンテナの停止
----------------------------------------------
実行しているコンテナを終了する

.. code-block:: console

  $ docker container stop コンテナIDまたはコンテナ名

- コンテナ名 ``fumi45`` のコンテナを終了する。

  .. code-block:: console

    $ docker container ls
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    db029554bc5f        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32769->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23
    $ docker container stop fumi45
    fumi45
    $ docker container ls
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23


2.3.5 docker container restart --- コンテナの再起動
---------------------------------------------------
一度停止したコンテナは破棄しない限り、再実行できる。

.. code-block:: console

  $ docker container restart コンテナIDまたはコンテナ名

- さっき停止した fumi45 を再実行する

  .. code-block:: console

    $ docker container restart fumi45
    fumi45
    $ docker container ls
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    db029554bc5f        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 3 seconds        0.0.0.0:32770->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23


2.3.6 docker container rm --- コンテナの破棄
--------------------------------------------
停止したコンテナをディスクから完全に破棄する。 (破棄しない限りはどんどん溜まる)

.. code-block:: console

  $ docker container rm コンテナIDまたはコンテナ名


- コンテナID ``4864fcaf1080`` のコンテナを破棄する。

  .. code-block:: console

    $ docker container ls -a
    CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                    PORTS                     NAMES
    db029554bc5f        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 days                 0.0.0.0:32769->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 days                 0.0.0.0:32768->8080/tcp   fumi23
    4864fcaf1080        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               gihyo-echo
    77699bc8d7cd        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               modest_saha
    $ docker container rm 4864fcaf1080
    4864fcaf1080
    $ docker container ls -a
    CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                    PORTS                     NAMES
    db029554bc5f        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 minutes              0.0.0.0:32770->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 days                 0.0.0.0:32768->8080/tcp   fumi23
    77699bc8d7cd        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               modest_saha

- 実行中のコンテナを停止・削除する。

  .. code-block:: console

    $ docker container ls
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    db029554bc5f        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 8 minutes        0.0.0.0:32770->8080/tcp   fumi45
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23
    $ docker container rm -f db029554bc5f
    db029554bc5f
    $ docker container ls
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23


停止の際にコンテナを破棄する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

  $ docker container run --rm

- コマンドラインツールとして利用するときなどに便利
- 停止したあとディスクに保持し続ける必要がないときに利用する
- 例)

  .. code-block:: console

    $ echo '{"version": 100}' | docker container run -i --rm gihyodocker/jq:1.5 '.version'
    100


2.3.7 docker container logs --- 標準出力の取得
-----------------------------------------------
実行している特定のコンテナの標準出力を表示する。

.. code-block:: console

  $ docker container logs [options] コンテナIDまたはコンテナ名

- 標準出力されているものだけが表示される。
- コンテナ内でアプリケーションがファイルに出力したようなログは表示されない


2.3.8 docker container exec --- 実行中コンテナでのコマンド実行
--------------------------------------------------------------
実行中の Docker コンテナの中で、任意のコマンドを実行できる。

.. code-block:: console

  $ docker container exec [options] コンテナIDまたはコンテナ名 コンテナ内で実行するコマンド


- 実行中のコンテナ ``fumi23`` 内で ``pwd`` コマンドを実行する。

  .. code-block:: console

    $ docker container exec fumi23 pwd
    /go

- コンテナをシェル経由で操作する。

  .. code-block:: console

    $ docker container exec -it fumi23 sh
    # pwd
    /go
    # exit


  - 本番環境ではやらないほうがよい


2.3.9 docker container cp --- ファイルのコピー
----------------------------------------------
コンテナ間、コンテナ・ホスト間でファイルをコピーできる。

.. code-block:: console

  $ docker container cp [options] コンテナIDまたはコンテナ名:コンテナ内のコピー元 ホストのコピー先

.. code-block:: console

  $ docker container cp [options] ホストのコピー元 コンテナIDまたはコンテナ名:コンテナ内のコピー先

- 実行中のコンテナ ``fumi23`` からホストのカレントディレクトリに ``main.go`` をコピーする。

  .. code-block:: console

    $ docker container cp fumi23:/echo/main.go .

- ホストのカレントディレクトリから、実行中のコンテナ ``fumi23`` に ``dummy.txt`` をコピーする。

  .. code-block:: console

    $ docker container cp dummy.txt fumi23:tmp
    $ docker container exec fumi23 ls /tmp | grep dummy
    dummy.txt

.. note::

  - コンテナ内で生成されたファイルをホストにコピーしてきて確認するようなデバッグ用途で使ったりする。
  - 破棄されていない終了したコンテナに対しても実行できる。


2.4 運用管理向けコマンド
=========================

2.4.1 prune --- 破棄
--------------------
停止しているコンテナを一括で削除する。

.. code-block:: console

  $ docker container prune [options]


- 途中で 確認を求められるので ``y`` と回答する。

  .. code-block:: console

    $ docker container ls -a
    CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                    PORTS                     NAMES
    81e3a724ae7c        example/echo:latest       "go run /echo/main.go"   5 days ago          Up 5 days                 0.0.0.0:32768->8080/tcp   fumi23
    77699bc8d7cd        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               modest_saha
    9a5c3a822e39        example/echo:latest       "go run /echo/main.go"   5 days ago          Exited (2) 5 days ago                               inspiring_goldstine
    f4e8a963eae4        example/echo:latest       "go run /echo/main.go"   5 days ago          Created                                             affectionate_curie
    b113261a42b8        example/echo:latest       "go run /echo/main.go"   6 days ago          Exited (2) 5 days ago                               ecstatic_tesla
    449ccdc8c99e        example/echo:latest       "go run /echo/main.go"   6 days ago          Exited (2) 6 days ago                               determined_zhukovsky
    a11a7535307a        example/echo:latest       "go run /echo/main.go"   6 days ago          Exited (2) 6 days ago                               vibrant_borg
    b8c42ba791e7        294c33d2b845              "go run /echo/main.go"   6 days ago          Exited (2) 6 days ago                               admiring_lalande
    9cd48659badb        gihyodocker/echo:latest   "go run /echo/main.go"   7 days ago          Exited (2) 7 days ago                               dreamy_saha
    fe9ad59901bb        gihyodocker/echo:latest   "go run /echo/main.go"   5 weeks ago         Exited (255) 7 days ago   0.0.0.0:9000->8080/tcp    vigilant_snyder
    $ docker container prune
    WARNING! This will remove all stopped containers.
    Are you sure you want to continue? [y/N] y
    Deleted Containers:
    77699bc8d7cd6992526da9171db5d10b511f46f4b12b8d68706825fddf8b7a18
    9a5c3a822e39ee5f811f21634c38cd4918a35e2e1ca0f680d170576fe98e7f33
    f4e8a963eae40f539e92b95b14236af8e614977d20bd80d11e0f870e6bfcdb0c
    b113261a42b8fb110cd1984904dccfe859067abd078637ff37804ad5f00c3ff5
    449ccdc8c99e72ecd791b036417632ec3e7944f1e7ab14c5b96d7e4caec0e58b
    a11a7535307a23a8121c7ac241e4df40f125a3187f556451e9014aa4f710046f
    b8c42ba791e7f266451bddc6d74b1eb196bf3b55d072bc8ff2f64a7f9c096648
    9cd48659badb6c5e8add684becbc91bae8fbeb6a928ae93618d8ff9fe3d36a6d
    fe9ad59901bbdd5dbad274eddc2def85fc49361a6299a7ae02f7693944c928ef

    Total reclaimed space: 29.41MB
    $ docker container ls -a
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
    81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   5 days ago          Up 5 days           0.0.0.0:32768->8080/tcp   fumi23


Docker イメージを一括削除する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Docker イメージも一括で削除できる。

.. code-block:: console

  $ docker image prune [options]

- 途中で 確認を求められるので ``y`` と回答する。

  .. code-block:: console

    $ docker image ls
    REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
    example/echo                     0.1.0               ed899b24590f        6 days ago          750MB
    example/echo                     latest              ed899b24590f        6 days ago          750MB
    fumi23/echo                      latest              ed899b24590f        6 days ago          750MB
    <none>                           <none>              294c33d2b845        7 days ago          750MB
    jenkins                          latest              cd14cecfdb3a        3 months ago        696MB
    golang                           1.9                 ef89ef5c42a9        3 months ago        750MB
    gihyodocker/jq                   1.5                 fb12c33cec33        10 months ago       5.31MB
    gihyodocker/echo                 latest              3dbbae6eb30d        10 months ago       733MB
    $ docker image prune
    WARNING! This will remove all dangling images.
    Are you sure you want to continue? [y/N] y
    Deleted Images:
    deleted: sha256:294c33d2b8454edba3e291fff2e2e477b287df30c13734a72fd8018cc4b4be9b
    deleted: sha256:73db87b05d43898a40665c4a8614bb383fc6bf050a37601e29da0fdb3f71e724
    deleted: sha256:8b7b14181869de8ccf721f5fc57b37b9d9ff533dc4c67201ff7b78862a67553c

    Total reclaimed space: 395B
    $ docker image ls
    REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
    example/echo                     0.1.0               ed899b24590f        6 days ago          750MB
    example/echo                     latest              ed899b24590f        6 days ago          750MB
    fumi23/echo                      latest              ed899b24590f        6 days ago          750MB
    jenkins                          latest              cd14cecfdb3a        3 months ago        696MB
    golang                           1.9                 ef89ef5c42a9        3 months ago        750MB
    gihyodocker/jq                   1.5                 fb12c33cec33        10 months ago       5.31MB
    gihyodocker/echo                 latest              3dbbae6eb30d        10 months ago       733MB


  - 残っているイメージは、 Docker が自動で判断して残しているもの。実行中のコンテナのイメージであるなど理由がある。


Docker リソースを一括削除する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
利用されていない Docker コンテナやイメージ、ボリューム、ネットワークといった全ての Docker リソースを一括で削除する。

.. code-block:: console

  $ docker system prune [options]


2.4.2 docker container stats --- 利用状況の取得
-----------------------------------------------
コンテナ単位でシステムリソースの利用状況を取得する。

.. code-block:: console

  $ docker container stats [options] [表示するコンテナID...]

- 実行例

.. code-block:: console

  $ docker container stats
  CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
  81e3a724ae7c        fumi23              0.00%               8.012MiB / 1.952GiB   0.40%               19.1kB / 0B         1.21MB / 8.19kB     17


2.5 Docker Compose でマルチコンテナを実行する
=============================================
- Docker コンテナ = 単一のアプリケーションと言い換えることができる
- 仮想サーバとっは対象とする粒度が異なる
- 複数存在するコンテナ同士が通信し、かつ、コンテナがコンテナの依存関係を持つはず

  - コンテナの挙動を制御するための設定ファイルや環境変数の与え方
  - コンテナ同士の依存関係
  - ポートフォワーディング


2.5.1 docker-compose によるコンテナの実行
-----------------------------------------
Compose: yaml 形式の設定ファイルで、複数のコンテナ実行を一括で管理できる。

- Docker コマンドで行なっていたコンテナの実行構成を設定ファイルで管理できるようになる


使えるかどうか確認する
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

  $ docker-compose version
  docker-compose version 1.22.0, build f46880f
  docker-py version: 3.4.1
  CPython version: 3.6.4
  OpenSSL version: OpenSSL 1.0.2o  27 Mar 2018


docker-compose でコンテナを実行する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. docker-compose.yml を用意する。

    .. code-block:: yaml

      # ===========================================================
      # $ docker container run -d -p 9000:8080 example/echo:latest
      # と同等の振る舞いを docker-compose で定義する
      # ===========================================================
      # docker-compose.yml ファイルフォーマットのバージョンを宣言
      version: "3"
      services:
        echo:  # コンテナの名前の定義
          # ここから下は実行するコンテナの定義
          image: example/echo:latest  # 使用する Docker イメージ
          ports:                      # ポートフォワーディングを指定
            - 9000:8080


2. docker-compose.yml を作成したディレクトリで実行する。

    .. code-block:: bash

      # コンテナ群を実行する
      $ docker-compose up -d
      Creating work_echo_1 ... done

      # 実行中のコンテナを一覧表示する
      $ docker container ls
      CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
      6287a2890ccb        example/echo:latest   "go run /echo/main.go"   8 seconds ago       Up 8 seconds        0.0.0.0:9000->8080/tcp    work_echo_1
      81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   6 days ago          Up 6 days           0.0.0.0:32768->8080/tcp   fumi23

      # コンテナを停止・削除する
      $ docker-compose down
      Stopping work_echo_1 ... done
      Removing work_echo_1 ... done
      Removing network work_default


docker-compose で Docker イメージをビルドして、そのまま実行する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. docker-compose.yml を用意する。

    .. code-block:: yaml

      # ===========================================================
      # docker-compose で、イメージのビルドと実行を一緒にやる
      # ===========================================================
      version: "3"
      services:
        echo:  # コンテナの名前の定義
          # ここ↓から実行するコンテナの定義
          build: .                    # Dockerfile が存在するディレクトリの相対パスを指定する
          ports:                      # ポートフォワーディングを指定する
            - 9000:8080


2. docker-compose.yml があるディレクトリで実行する。

    .. code-block:: bash

      # コンテナ群を実行する
      # --build: up 時に Docker イメージを必ずビルドするオプション
      $ docker-compose up -d --build
      Creating network "echo_default" with the default driver
      Building echo
      Step 1/4 : FROM golang:1.9
       ---> ef89ef5c42a9
      Step 2/4 : RUN mkdir /echo
       ---> Using cache
       ---> 7caf124fb4d3
      Step 3/4 : COPY main.go /echo
       ---> 15b6deaeb7ce
      Step 4/4 : CMD ["go", "run", "/echo/main.go"]
       ---> Running in a9b3b311fdc9
      Removing intermediate container a9b3b311fdc9
       ---> 291b78994229
      Successfully built 291b78994229
      Successfully tagged echo_echo:latest
      Creating echo_echo_1 ... done

      # 起動したことを確認する
      $ docker container ls
      CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
      6f0571015a21        echo_echo             "go run /echo/main.go"   39 seconds ago      Up 38 seconds       0.0.0.0:9000->8080/tcp    echo_echo_1
      81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   6 days ago          Up 6 days           0.0.0.0:32768->8080/tcp   fumi23


2.6 Compose による複数コンテナの実行
====================================

2.6.1 Jenkins コンテナを実行する
--------------------------------

1. ``docker-compose.yml``

    .. code-block:: yaml

      # ================================
      # 2.6.1 Jenkins コンテナを実行する
      # ================================
      version: "3"
      services:
        master:  # コンテナの名前
          # 実行するコンテナの定義
          # ======================
          container_name: master
          image: jenkins:latest      # Docker Hub に登録されている Jenkins の公式イメージを利用する
          ports:                      # ポートフォワーディングを指定する
            - 9000:8080
          volumes:                    # ホスト・コンテナ間でファイルを共有する仕組み
            - ./jenkins_home:/var/jenkins_home    # ホストの `./jenkins_home` に、Jenkins コンテナの `/var/jenkins_home` をマウント

2. Compose で実行する。

    .. code-block:: bash

      # docker-compose.yml のあるディレクトリで実行する
      $ docker-compose up

3. ``http://localhost:9000/`` にアクセスして Jenkins の初期設定をする。


2.6.2 Master Jenkins の SSH 鍵を作る
------------------------------------

.. code-block:: bash

  $ docker container exec -it master ssh-keygen -t rsa -C ""
  Generating public/private rsa key pair.
  Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa):
  Created directory '/var/jenkins_home/.ssh'.
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in /var/jenkins_home/.ssh/id_rsa.
  Your public key has been saved in /var/jenkins_home/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:Ee3KESkiYjH4QoZBZqPie9PH5rZ9GsVBfUaUAuXwXaY
  The key's randomart image is:
  +---[RSA 2048]----+
  |=O.     .o ++.oo+|
  |B++ . . o.o +o.*.|
  |*o . . ..o . oE. |
  |+ .     ..o .    |
  | o     .So o     |
  |  . . . o .      |
  | . o . + .       |
  |  . . +.. ..     |
  |      .o.oo      |
  +----[SHA256]-----+


2.6.3 Jenkins Slave コンテナを作る
----------------------------------

1. ``docker-compose.yml``

    .. code-block:: yaml

      # ===================================
      # 2.6.3 Jenkins Slave コンテナを作る
      # ===================================
      version: "3"
      services:
        # Master コンテナ
        master:
          container_name: master
          image: jenkins:latest       # Docker Hub に登録されている Jenkins の公式イメージを利用する
          ports:                      # ポートフォワーディングを指定する
            - 9000:8080
          volumes:                    # ホスト・コンテナ間でファイルを共有する仕組み
            - ./jenkins_home:/var/jenkins_home    # ホストの `./jenkins_home` に、Jenkins コンテナの `/var/jenkins_home` をマウント
          links:
            - slave01                 # これで、 master から slave01 で名前解決できるようになる
          # pw: bb91ef8ec2e74cc3a99802a79a84df6b

        # Slave コンテナ
        slave01:
          container_name: slave01
          image: jenkinsci/ssh-slave    # SSH 接続する Slave 用途の Docker イメージを利用する
          environment:
            - JENKINS_SLAVE_SSH_PUBKEY=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCd2Nn7oT9F9JoWCWuTyjkmADAMrNZzZGCBZjT118QD3518CsaTlLmE8ikqloO9rCi58Vgi9nBhSJ0SGGG+Vx8mBZlG5V/ucq3TlbHq9Epdr8mJAaJ44F3CWNso97WIBzDEdtCAYQPRO1kEjlGBOr2JAAkYPyNEDe9o2Ta02FzqLu4WpqmqopgS6xsDWqj3BfQoJ+MCLeej865zSETyFojz8BbYH03LMhzBUeb6Yfa9LtIJgpp0KMab9p3hKlV5Es6jIVhYlVBw2NokWkOaq7KjlwTj0pmUGYIBNpgg+POpkaMt42dVNiqYxvbEzz9ZaejM3T/DIORIhKtYnnANwN4b
            # ↑これで Master からの SSH 接続が許可された状態になる
            # Slave コンテナの `~/.ssh/authorized_keys ` ファイルに Master コンテナの SSH 公開鍵が追加される。


2. Compose で実行する。

    .. code-block:: console

      $ docker-compose up -d
      Creating slave01 ... done
      Creating master  ... done
      $ docker-compose ps
       Name                Command               State                 Ports
      ------------------------------------------------------------------------------------
      master    /bin/tini -- /usr/local/bi ...   Up      50000/tcp, 0.0.0.0:9000->8080/tcp
      slave01   setup-sshd                       Up      22/tcp
      (venv) FumienoMacBook-Pro:2.6.1 fumi23$ docker container ls -a
      CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                    PORTS                               NAMES
      91ede878960a        jenkins:latest        "/bin/tini -- /usr/l…"   42 minutes ago      Up 42 minutes             50000/tcp, 0.0.0.0:9000->8080/tcp   master
      5fc916903044        jenkinsci/ssh-slave   "setup-sshd"             42 minutes ago      Up 42 minutes             22/tcp                              slave01
      6f0571015a21        echo_echo             "go run /echo/main.go"   6 days ago          Exited (2) 15 hours ago                                       echo_echo_1
      81e3a724ae7c        example/echo:latest   "go run /echo/main.go"   12 days ago         Exited (2) 15 hours ago                                       fumi23
      $ docker container exec master ls /usr/share/jenkins
      jenkins.war
      ref

3. Jenkins のバージョンが古く、SSH Slaves plugin が使えなかったので、 Docker コンテナ内の Jenkins をバージョンアップする。

    .. code-block:: console

      $ docker container exec -u 0 -it master bash
      root@91ede878960a:/# wget http://updates.jenkins-ci.org/download/war/2.138.3/jenkins.war
      --2018-11-18 06:21:47--  http://updates.jenkins-ci.org/download/war/2.138.3/jenkins.war
      Resolving updates.jenkins-ci.org (updates.jenkins-ci.org)... 52.202.51.185
      Connecting to updates.jenkins-ci.org (updates.jenkins-ci.org)|52.202.51.185|:80... connected.
      HTTP request sent, awaiting response... 302 Found
      Location: http://mirrors.jenkins-ci.org/war-stable/2.138.3/jenkins.war [following]
      --2018-11-18 06:21:47--  http://mirrors.jenkins-ci.org/war-stable/2.138.3/jenkins.war
      Resolving mirrors.jenkins-ci.org (mirrors.jenkins-ci.org)... 52.202.51.185
      Reusing existing connection to updates.jenkins-ci.org:80.
      HTTP request sent, awaiting response... 302 Found
      Location: http://ftp.yz.yamagata-u.ac.jp/pub/misc/jenkins/war-stable/2.138.3/jenkins.war [following]
      --2018-11-18 06:21:48--  http://ftp.yz.yamagata-u.ac.jp/pub/misc/jenkins/war-stable/2.138.3/jenkins.war
      Resolving ftp.yz.yamagata-u.ac.jp (ftp.yz.yamagata-u.ac.jp)... 133.24.248.18, 133.24.248.16, 133.24.248.19, ...
      Connecting to ftp.yz.yamagata-u.ac.jp (ftp.yz.yamagata-u.ac.jp)|133.24.248.18|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 75733340 (72M)
      Saving to: ‘jenkins.war’

      jenkins.war                                        100%[===============================================================================================================>]  72.22M  3.97MB/s    in 14s

      2018-11-18 06:22:01 (5.30 MB/s) - ‘jenkins.war’ saved [75733340/75733340]

      root@91ede878960a:/# ls -la
      total 74032
      drwxr-xr-x   1 root root     4096 Nov 18 06:21 .
      drwxr-xr-x   1 root root     4096 Nov 18 06:21 ..
      -rwxr-xr-x   1 root root        0 Nov 18 05:03 .dockerenv
      drwxr-xr-x   1 root root     4096 Jul 17 16:20 bin
      drwxr-xr-x   2 root root     4096 Jun 26 12:03 boot
      drwxr-xr-x   5 root root      340 Nov 18 05:03 dev
      lrwxrwxrwx   1 root root       33 Jul 17 06:16 docker-java-home -> /usr/lib/jvm/java-8-openjdk-amd64
      drwxr-xr-x   1 root root     4096 Nov 18 05:03 etc
      drwxr-xr-x   2 root root     4096 Jun 26 12:03 home
      -rw-r--r--   1 root root 75733340 Nov  9 01:03 jenkins.war
      drwxr-xr-x   1 root root     4096 Jul 16 00:00 lib
      drwxr-xr-x   2 root root     4096 Jul 16 00:00 lib64
      drwxr-xr-x   2 root root     4096 Jul 16 00:00 media
      drwxr-xr-x   2 root root     4096 Jul 16 00:00 mnt
      drwxr-xr-x   2 root root     4096 Jul 16 00:00 opt
      dr-xr-xr-x 187 root root        0 Nov 18 05:03 proc
      drwx------   2 root root     4096 Jul 16 00:00 root
      drwxr-xr-x   3 root root     4096 Jul 16 00:00 run
      drwxr-xr-x   1 root root     4096 Jul 17 03:13 sbin
      drwxr-xr-x   2 root root     4096 Jul 16 00:00 srv
      dr-xr-xr-x  13 root root        0 Nov 18 05:03 sys
      drwxrwxrwt   1 root root     4096 Nov 18 05:04 tmp
      drwxr-xr-x   1 root root     4096 Jul 16 00:00 usr
      drwxr-xr-x   1 root root     4096 Jul 17 16:20 var
      root@91ede878960a:/# mv ./jenkins.war /usr/share/jenkins
      root@91ede878960a:/# chown jenkins:jenkins /usr/share/jenkins/jenkins.war
      root@91ede878960a:/# exit
      exit
      $ docker-compose restart
      Restarting master  ... done
      Restarting slave01 ... done

4. ``slave01`` ノードを追加して、 SSH の設定をする。


参考サイト
^^^^^^^^^^
ありがとうございました!!
- https://medium.com/@jimkang/how-to-start-a-new-jenkins-container-and-update-jenkins-with-docker-cf628aa495e9
