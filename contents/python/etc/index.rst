.. article::
   :date: 2018-10-11
   :title: Python いろいろメモ
   :category: python
   :tags:
   :canonical_url: /python/etc/
   :draft: false


===================
Python いろいろメモ
===================


wheel
=====
- ビルド済みの C 拡張や Python パッケージのみを含み、ファイルを展開するだけでインストールが完了する
- sdist : パッケージのソース、メタデータ、ビルド方法などをアーカイブしたソース配布形式

  - インストールのたびに各環境でアーカイブに同梱される setup スクリプトを読み込み、 C 拡張があればビルドし、必要は Python パッケージを確認して、 site-packages へコピーする

- Python の公式バイナリパッケージは wheel 形式 (PEP491)
- pip コマンドは、 wheel 形式を優先して利用する
- pip は、 PyPI にアップロードされている wheel 形式のパッケージを直接インストールできる

  .. code-block:: console

    $ pip install django==1.11.15

- https://pythonwheels.com/ : 定番パッケージの wheel 配布状況を確認できる


wheel の作り方
--------------
PyPI で sdist で配布されているパッケージを wheel 形式のパッケージに変換してローカルに保存する。

1. wheel をインストールする

    .. code-block:: console

      (venv) $ pip install wheel

2. wheelhouse ディレクトリに wheel 形式パッケージを作成する

    .. code-block:: console

      (venv) $ pip wheel markupsafe -w wheelhouse

    - ``wheelhouse`` ディレクトリがなくても、 ``-w wheelhouse`` を指定すると勝手に作ってくれる
    - wheel 形式のパッケージの保存ディレクトリ名は何でもよいが、慣習的に ``wheelhouse`` という名前を使う


参考
^^^^
Python プロフェッショナルプログラミング 第3版 P.256 - P.259


pip
====

pip のリファレンス
------------------
https://pip.pypa.io/en/stable/reference/


pip install -r
------------------

requirements.txt に指定したライブラリをインストールする

.. code-block:: console

  $ pip install -r requirements.txt


- すでにインストール済みのものはスキップしてくれる


- requirements.txt にインストールオプションを書いておくことができる

  .. code-block:: console

    --no-index           # PyPI に問い合わせない (Index サーバーを使わない)
    -f wheelhouse        # ライブラリの取得元を wheelhouse に限定する
    -r run-requires.txt  # インストールしたいライブラリはこっちに書いたから見てね

  - ``-f(--find-links) <url>`` : 参照したいパッケージがあるページのリンクを指定する。

    - url に存在するパッケージは、 Index サーバーよりも優先的に使用される
    - url に見つからないパッケージは、 Index サーバーからインストールする


参考
^^^^
Python プロフェッショナルプログラミング 第3版 P.272 - P.274, P.255


pip install -U
------------------
最新のバージョンに更新する

  .. code-block:: console

    $ pip install -U requests


  - pip は、指定されたパッケージがすでにインストール済みの場合、新しいバージョンが公開されていても自動的に最新版に更新したりしない


参考
^^^^
Python プロフェッショナルプログラミング 第3版 P.63


rundeckrun
==========
Python コードから Rundeck を操作できる。

リファレンス
------------
https://rundeckrun.readthedocs.io/en/latest/index.html


autopep8
========
`autopep8 automatically formats Python code to conform to the PEP 8 style guide.`

- https://pypi.python.org/pypi/autopep8


pipdeptree
==========
ライブラリの依存関係を調べられる。

- https://github.com/naiquevin/pipdeptree

  .. code-block:: console

    $ pip install pipdeptree
    $ pipdeptree -p django
    Django==1.11.15
      - pytz [required: Any, installed: 2018.3]


  .. code-block:: console

    $ pipdeptree -r -p django
    django==1.11.15
      - dj-inmemorystorage==1.4.1 [requires: Django>=1.4]
      - model-mommy==1.5.1 [requires: django>=1.8.0]


  - オプションの意味

    .. code-block:: console

      -r, --reverse         Shows the dependency tree in the reverse fashion ie.
                            the sub-dependencies are listed with the list of
                            packages that need them under them.
      -p PACKAGES, --packages PACKAGES
                            Comma separated list of select packages to show in the
                            output. If set, --all will be ignored.
