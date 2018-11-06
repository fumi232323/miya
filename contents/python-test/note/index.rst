.. article::
   :date: 2018-11-06
   :title: Python テストのメモ
   :category: test
   :tags:
   :canonical_url: /python-test/note/
   :draft: false


===================
Python テストのメモ
===================


mypy
====

リファレンス
------------
https://mypy.readthedocs.io/en/latest/index.html

Type hints 覚書き
-----------------

:Union[int, str]: ``int`` or ``str`` の意。とり得る型が全部わかっているなら、 ``Any`` と書くより列記したほうがよい。
:Optional[str]: ``str`` or ``None`` の意
:Any: なんでもいい
:ignore: mypy エラーを抑止

- Any は型チェックの敗北
- ignore は敗北その2


いろいろ
--------
- ``実行時にわかること`` は mypy にはわからない
- mypy が解釈できないなら人間にも読みにくい


役に立つ情報
-------------
- `Python と型チェッカー <https://www.slideshare.net/t2y/python-typechecker-20180519>`_
- `Mypy 0.600 Released <http://mypy-lang.blogspot.com/2018/05/>`_


TODO
-----
- Type hints はどこに必要か、どこに書くべきか

  - 引数と戻り値だけ書いておくのかと思っていたけれど、違っていた
  - 定数の宣言 & 初期化時 (空の場合のみ) は書く
  - 変数でも書くときがある
  - たぶん、型を (強制はできないけど) 規制したいときは全部書けば良いんじゃないかと思う
  - 「typecheck エラーが出るから書く」んじゃなくて、まわりのひとに、こうしてほしい (この型にしてほしい) って知らせたいときに。


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

tox
---
これをテストに使いたいとは限らないのでもうテスト用に環境分けちゃおうぜ的なツール


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
