.. article::
   :date: 2018-10-01
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

- Docker イメージ

  - Docker コンテナを構成するファイルシステムや、実行するアプリケーションや設定をまとめたもの
  - コンテナを作成するために利用されるテンプレートとなるもの

- Docker コンテナ

  - Docker イメージを基に作成され、具現化されたファイルシステムとアプリケーションが実行されている状態

- ひとつの Docker イメージから複数のコンテナを生成できる
- まずは、イメージから作成する


2.1.1 Docker イメージと Docker コンテナの基本
---------------------------------------------

- Docker イメージをダウンロード

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


- Docker イメージを実行

  .. code-block:: console

    FumienoMacBook-Pro:docker-work fumi23$ docker container run -t -p 9000:8080 gihyodocker/echo:latest
    2018/10/01 13:53:03 start server

  - ポートフォワーディング設定をしている

    - Docker 実行環境の 9000 ポート経由で HTTP リクエストを受けられるようになっている

- 別のターミナルからアクセスしてみる

  .. code-block:: console

    FumienoMacBook-Pro:~ fumi23$ curl http://localhost:9000
    Hello Docker!!


  .. code-block:: console

    FumienoMacBook-Pro:docker-work fumi23$ docker container run -t -p 9000:8080 gihyodocker/echo:latest
    2018/10/01 13:53:03 start server
    2018/10/01 13:56:44 received request


- 停止する
