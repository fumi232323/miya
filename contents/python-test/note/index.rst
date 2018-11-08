.. article::
   :date: 2018-11-08
   :title: Python テストのメモ
   :category: test
   :tags:
   :canonical_url: /python-test/note/
   :draft: false


===================
Python テストのメモ
===================


tox
====
`これをテストに使いたいとは限らないのでもうテスト用に環境分けちゃおうぜ的なツール`

- 別々の virtualenv にそれぞれテスト環境を構築してくれる
- 構築した virtualenv の中でテストを実行して、結果を表示してくれる
- テストツールや環境変数などをまとめておける
- ``tox`` コマンドを実行するだけで、必要なテストを簡単に、まとめて実行できるようになる
- それぞれのテストは virtualenv によって分離されているため、他のテスト用設定が別のテストに影響を与えない


リファレンス
------------
- https://tox.readthedocs.io/en/latest/config.html
- https://docs.pytest.org/en/latest/reference.html#configuration-options


インストール
-------------

.. code-block:: console

  $ pip install tox


tox.ini
-------
テスト実行環境の設定を記述するファイル

.. code-block:: ini

  [tox]
  # envlist: テスト環境の一覧。それぞれ別々の環境が用意される。
  # py27: インストールされている python2.7 コマンドを探し出して、 Python2.7 の virtualenv を作成してくれる。
  envlist = flake8,typecheck,py27

  # setup.py がなくても実行可能にする。 setup.py がない場合は、 requirements.txt などで依存ライブラリを管理する。
  skipsdist = True

  [pytest]
  addopts = --durations=10

  # 実行対象を指定する。この場合は、 tests ディレクトリ配下。
  testpaths = tests

  # pytest-pythonpath の項を参照のこと。
  python_paths = apps tests

  django_find_project = false
  DJANGO_SETTINGS_MODULE = settings.test

  # [testenv]: テスト環境の設定。
  # `[testenv:flake8]` のように環境の名前が指定されている場合は、そちらの指定が優先される。
  # 環境の名前に合う設定がない場合は、 `[testenv]` セクションの汎用的な設定が利用される。
  [testenv]

  # 環境にインストールするライブラリを指定する。
  # この部分は pip に直接渡されるため、ライブラリではなく requirements.txt の指定も可能。
  # `-r` と `requirements_dev.txt` の間にスペースを入れるとエラーになるので注意。
  deps = -rrequirements_dev.txt

  # 環境変数を固定したい場合はここに書く
  setenv =
      PYTHONDONTWRITEBYTECODE = 1

  # 実行するコマンド: py.test, 実行対象: tox コマンド実行時に引数として渡されたディレクトリ配下
  # `{posargs}` と書くと、 tox コマンド実行時に引数を渡すことができる
  commands = py.test {posargs}

  # typecheck はこちらのテスト環境設定が優先される
  [testenv:typecheck]
  # typecheck 用の環境にインストールするライブラリを指定する。
  deps =
      mypy
      mypy-extensions

  # typecheck 用の virtualenv は python3 で作る
  # https://tox.readthedocs.io/en/latest/config.html#conf-basepython
  basepython = python3

  # 実行するコマンド: mypy, 実行対象: app ディレクトリ配下
  commands = mypy apps

  # flake8 はこちらのテスト環境設定が優先される
  [testenv:flake8]
  deps =
      flake8>=3.6.0
      flake8-blind-except
      flake8-docstrings<1.1.0
      flake8-import-order
      pydocstyle<2.0.0
      mccabe
      radon

  # https://docs.python.org/ja/3/using/cmdline.html#envvar-PYTHONDONTWRITEBYTECODE (よくわからない)
  setenv =
      PYTHONDONTWRITEBYTECODE = 1

  # 実行するコマンド: flake8, 実行対象: app ディレクトリ配下
  commands = flake8 apps

  [flake8]
  # 実行対象外リスト。除外するディレクトリを列記する。
  # `[testenv:flake8]` セクションに `app` 配下を実行対象とする、と書いてあるので、 `app` の中の `migrations,urls.py,manage.py,settings` 配下は対象外、の意。
  exclude = migrations,urls.py,manage.py,settings

  max-line-length = 120
  max-complexity = 10
  radon-max-cc = 10
  import-order-style = google

  # flake8 警告を抑止するリスト。詳しくは shihmizukawa さんの scrapbox ↓ を参照のこと。
  extend-ignore = C901,D100,D101,D102,D103,D104,D105,D200,D202,D203,D204,D205,D208,D209,D210,D300,D301,D302,D400,D401,D402,D403,E741,I100,I101,R701


実行
----

全部実行する。 tox.ini ファイルのあるディレクトリで実行する!!

.. code-block:: console

  $ tox

``-e`` オプションを指定すると、指定した環境のテストのみが実行できる。

.. code-block:: bash

  # pytest だけ
  $ tox -e py27
  # pytest だけ, tests/test_target 配下だけ
  $ tox -e py27 tests/test_target
  # flake8 だけ
  $ tox -e flake8
  # flake8 と typecheck
  $ tox -e flake8, typecheck


tox ではなく、テストコマンドにオプションを渡したい場合は、 ``--`` のあとにオプションを指定する。

.. code-block:: console

  $ tox -e py27 -- -vv tests/test_target


テスト用仮想環境の再作成
------------------------

.. code-block:: console

  $ tox -r

- tox.ini から参照している requirements.txt の中身を変更したあとは、明示的にテスト用仮想環境を再作成する必要がある。

  - tox は、 テスト用の仮想環境を作成するときに、 ``-rrequirements_dev.txt`` 引数を内部で pip コマンドに渡して実行する。

    - tox.ini の ``deps`` の記述が更新された場合は、テスト用の仮想環境を再作成してくれる。
    - ``requirements_dev.txt`` の中身だけ更新されて、``deps`` 自体の更新がない場合は、 仮想環境の再作成も pip コマンドの再実行も行わない。


pytest-pythonpath
-----------------
テスト実行の前に、 pytests.ini に指定した検索パスを ``PYTHONPATH`` に追加してくれるプラグイン

  - 使い方はこちら: https://pypi.org/project/pytest-pythonpath/
  - ``PYTHONPATH`` はこちら: https://docs.python.org/ja/3/using/cmdline.html#envvar-PYTHONPATH


参考書籍, サイト
-----------------
- Python プロフェッショナル プログラミング 第3版: P.249, 274
- http://note.crohaco.net/2016/python-tox/
- http://note.crohaco.net/2016/python-pytest/
- https://tox.readthedocs.io/en/latest/config.html#conf-basepython
- https://docs.python.org/ja/3/using/cmdline.html#envvar-PYTHONDONTWRITEBYTECODE
- https://pypi.org/project/pytest-pythonpath/
- https://docs.python.org/ja/3/using/cmdline.html#envvar-PYTHONPATH


flake8 3.6 対応がわかりやすい
=============================
https://scrapbox.io/shimizukawa/flake8-3.6.0_%E3%81%AB%E6%9B%B4%E6%96%B0%E3%81%97%E3%81%9F%E3%82%89%E8%AD%A6%E5%91%8A%E3%81%9F%E3%81%8F%E3%81%95%E3%82%93%E5%87%BA%E3%81%A6%E3%81%8D%E3%81%9F


mypy
====

リファレンス
------------
https://mypy.readthedocs.io/en/latest/index.html


Type hints 覚書き
-----------------
- `Type hints cheat sheet (Python 3) <https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html>`_
- `Type hints cheat sheet (Python 2) <https://mypy.readthedocs.io/en/latest/cheat_sheet.html>`_

:Union[int, str]: ``int`` or ``str`` の意。とり得る型が全部わかっているなら、 ``Any`` と書くより列記したほうがよい。
:Optional[str]: ``str`` or ``None`` の意
:Any: なんでもいい
:ignore: mypy エラーを抑止


これ
----
- `Any は型チェックの敗北`
- `ignore は敗北その2`
- ``実行時にわかること`` は mypy にはわからない
- `mypy が解釈できないなら人間にも読みにくい`


Type hints はどこに必要か、どこに書くべきか
-------------------------------------------
- 引数と戻り値
- 定数の初期化時 (空の場合のみ)
- 変数の初期化時
- まわりのひとに、こうしてほしい (この型にしてほしい) って知らせたいとき
- ついていたらありがたいと (自分が) 思うところ
- どこという決まりはどうもないらしい (調べていて今のところ見つかっていない)
- とりあえずのところ、 mypy に怒られるところにつけていけば、そのうちわかるようになると思う


python3 ではこういうふうに書ける
--------------------------------

.. code-block:: python

  # https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html

  age: int = 1
  x: List[int] = [1]

  def f(num1: int, my_float: float = 3.5) -> float:
    return num1 + my_float


- `typing --- 型ヒントのサポート <https://docs.python.org/ja/3/library/typing.html>`_ ( `バージョン 3.5 で追加` と言っている)
- python3 でも、type hints を書いたからと言って、言語側でチェックしてくれるわけではない。 `static type checker` と言っているのが、 ``mypy`` とかのこと。


Tips
-----
- PyCharm で、 ``関数名やら定数やら変数やらを範囲選択`` -> ``Option + Enter`` で type hints を自動追加できる

  - ガイド: `PyCharmのヒント <https://pleiades.io/help/pycharm/type-hinting-in-product.html>`_
  - いくつか試してみたけど、半々くらいの確率でちゃんとなる (期待どおりの type hints が追加できる)


情報
----
- `Python と型チェッカー <https://www.slideshare.net/t2y/python-typechecker-20180519>`_
- `Mypy 0.600 Released <http://mypy-lang.blogspot.com/2018/05/>`_
- `Mypy: Getting started <https://mypy.readthedocs.io/en/latest/getting_started.html>`_

  - `Static types in Python, oh my(py)! <https://blog.zulip.org/2016/10/13/static-types-in-python-oh-mypy/>`_
  - `Carl Meyer - Type-checked Python in the real world - PyCon 2018 <https://www.youtube.com/watch?v=pMgmKJyWKn8>`_

- `PEP 484 -- Type Hints <https://www.python.org/dev/peps/pep-0484/>`_


pytest
======

例外のテストはこれ
------------------
https://docs.pytest.org/en/latest/reference.html#pytest-raises

- ``with pytest.raises(RuntimeError) as excinfo:`` の ``excinfo`` には、 `ExceptionInfo <https://docs.pytest.org/en/latest/reference.html#exceptioninfo>`_ が入ってくる
- 例外のインスタンスは、 ``value`` フィールドに入っている

  .. code-block:: python

      with pytest.raises(ListFileError) as e:
          target(list_file).read('201803')

      # assert
      assert e.value.line == 2


parametrize
-----------
``unittest.TestCase`` を継承したテストクラス内では、 ``@pytest.mark.parametrize`` を使えない。


pytest.fixture
--------------
https://docs.pytest.org/en/latest/fixture.html

::

  Test functions can receive fixture objects by naming them as an input argument. For each argument name, a fixture function with that name provides the fixture object. Fixture functions are registered by marking them with @pytest.fixture.


あれこれ
========

3a スタイル
-----------

.. code-block:: python

  # arrange
  # act
  # assert


http://wiki.c2.com/?arrangeactassert (つながらない)


TestPyramid
--------------
UI 寄りのテストは、コストと実行時間が長くなってしまうので、いきなり書かない方がいい

- https://martinfowler.com/bliki/TestPyramid.html


テストターゲットの取得方法
----------------------------
http://pelican.aodag.jp/xiao-guo-de-naunittest-mataha-callfutnomi-mi.html
