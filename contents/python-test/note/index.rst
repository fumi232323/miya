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


.. contents:: 目次


pytest
======

リファレンス
------------
https://docs.pytest.org/en/latest/

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


testfixtures
============

リファレンス
------------
https://testfixtures.readthedocs.io/en/latest/index.html


date と datetime を mock できる
--------------------------------

- https://testfixtures.readthedocs.io/en/latest/datetime.html
- https://testfixtures.readthedocs.io/en/latest/api.html#testfixtures.test_date
- https://testfixtures.readthedocs.io/en/latest/api.html#testfixtures.test_datetime

  - ``test_datetime`` は、 ``datetime.datetime.now()`` にしか効果を及ぼさないので注意!!
  - ``datetime.datetime.today()`` には何の効果もない


flake8 3.6 対応がわかりやすい
=============================
https://scrapbox.io/shimizukawa/flake8-3.6.0_%E3%81%AB%E6%9B%B4%E6%96%B0%E3%81%97%E3%81%9F%E3%82%89%E8%AD%A6%E5%91%8A%E3%81%9F%E3%81%8F%E3%81%95%E3%82%93%E5%87%BA%E3%81%A6%E3%81%8D%E3%81%9F


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
