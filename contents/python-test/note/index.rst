.. article::
   :date: 2018-10-11
   :title: Python テストのメモ
   :category: test
   :tags:
   :canonical_url: /python-test/note/
   :draft: false


=======================
Python テストのメモ
=======================


Python テストのメモ
=======================

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
--------------
``unittest.TestCase`` を継承したテストクラス内では、 ``@pytest.mark.parametrize`` を使えない。


tox
--------------
これをテストに使いたいとは限らないのでもうテスト用に環境分けちゃおうぜ的なツール


pytest.fixture
----------------------------
https://docs.pytest.org/en/latest/fixture.html

::

  Test functions can receive fixture objects by naming them as an input argument. For each argument name, a fixture function with that name provides the fixture object. Fixture functions are registered by marking them with @pytest.fixture.


3a スタイル
----------------------------

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
