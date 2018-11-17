.. article::
   :date: 2018-11-17
   :title: mypy
   :category: test
   :tags:
   :canonical_url: /python-test/mypy/
   :draft: false


====
mypy
====


.. contents::


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
