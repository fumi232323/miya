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

参考
^^^^
Python プロフェッショナルプログラミング 第3版 P.272 - P.274


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
