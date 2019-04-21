.. article::
   :date: 2019-04-20
   :title: Python のメモ
   :category: python
   :tags:
   :canonical_url: /python/note/
   :draft: false


==================
Python のメモ
==================


リファレンス
------------
- `頭文字別索引: <https://docs.python.org/ja/3/genindex.html>`_
- `Pythonモジュール索引 <https://docs.python.org/ja/3/py-modindex.html>`_
- `組み込み関数 <https://docs.python.org/ja/3/library/functions.html#built-in-functions>`_
- `Python 標準ライブラリ <https://docs.python.org/ja/3/library/index.html>`_


ローカルに HTTP サーバーを立てる
--------------------------------

.. code-block:: python

  # ポートを指定 (現在のディレクトリのファイルを提供)
  $ python -m http.server 8000
  # バインドするアドレスを指定 (現在のディレクトリのファイルを提供)
  $ python -m http.server 8000 --bind 127.0.0.1
  # ファイルを提供するディレクトリを指定
  $ python -m http.server --directory /tmp/


リファレンス
^^^^^^^^^^^^
`http.server --- HTTP サーバ <https://docs.python.org/ja/3/library/http.server.html>`_


ユーザー定義例外
----------------
- Exception クラスを、直接または間接的に継承する

  - https://docs.python.org/ja/3/library/exceptions.html#BaseException

    ::

      全ての組み込み例外の基底クラスです。ユーザ定義の例外に直接継承されることは意図されていません (継承には Exception を使ってください)。


- 大抵は、いくつかの属性だけを提供し、例外が発生したときにハンドラがエラーに関する情報を取り出せるようにする程度にとどめる
- だいたいは、標準の例外の名前付けと同様に、 "Error" で終わる名前で定義する
- 複数の別個の例外を送出するようなモジュールを作成する際には、そのモジュールで定義されている例外の基底クラスを作成するのが一般的なプラクティス


リファレンス
^^^^^^^^^^^^
`8.5. ユーザー定義例外 <https://docs.python.org/ja/3.7/tutorial/errors.html#user-defined-exceptions>`_


re.MatchObject.groupdict
------------------------
- マッチの、すべての 名前つきの サブグループを含む、サブグループ名でキー付けされた辞書を返す
- リファレンスのサンプルコードを見ると一目瞭然なので、そちらを見てください

リファレンス
^^^^^^^^^^^^
https://docs.python.org/ja/2.7/library/re.html#re.MatchObject.groupdict


どんどん足してくやつ
--------------------
- `functools.reduce <https://docs.python.org/ja/3/library/functools.html#functools.reduce>`_


coding: utf-8
-------------
ソースコードの文字エンコーディングを指定する

.. code-block:: python

  # -*- coding: utf-8 -*-

- ファイルの先頭に記述する
- 記述しないと、 Python2 環境かつファイルにマルチバイトが含まれていると ``SyntaxError`` が発生する。
- Python3 環境では不要


yield dict(zip(columns, data))
------------------------------
これは、

.. code-block:: python

  for column, value in zip(columns, data):
      row_dict[column] = value
      yield row_dict

こう書ける。

.. code-block:: python

  yield dict(zip(columns, data))


- ``zip`` はタプルのイテレータを返す -> タプルから辞書を作れる
- ``dict(iterable, **kwarg)``
- https://docs.python.org/ja/3/library/stdtypes.html#dict

  ::

    それ以外の場合、位置引数は iterable オブジェクトでなければなりません。iterable のそれぞれの要素自身は、ちょうど 2 個のオブジェクトを持つイテラブルでなければなりません。それぞれの要素の最初のオブジェクトは新しい辞書のキーになり、2 番目のオブジェクトはそれに対応する値になります。同一のキーが 2 回以上現れた場合は、そのキーの最後の値が新しい辞書での対応する値になります。


defaultdict
-----------
リストの初期化が不要になる！

- `defaultdict オブジェクト <https://docs.python.org/ja/3/library/collections.html#defaultdict-objects>`_
- `defaultdict の使用例 <https://docs.python.org/ja/3/library/collections.html#defaultdict-examples>`_


組み込み型と名前が被った場合
----------------------------
``in`` や ``int`` など、キーワード・組み込み型と同じ名前を変数名にしたい場合は、末尾に ``_`` を付ける。


sorted
------
これは、

.. code-block:: python

  summary_list = list(summary_dict.values())
  summary_list.sort(key=lambda x: x['sort_key'])


``sorted`` という関数を使って以下のように書ける。

.. code-block:: python

  summary_list = sorted(summary_dict.values(), key=lambda x: x['sort_key'])

さらに、for文をこんなふうに書くと ``summary_list`` を作る工程が不要。

.. code-block:: python

  for _, summary in sorted(summary_dict.items()):
      ....


リファレンス
^^^^^^^^^^^^
`タプルはイミュータブルなシーケンス` なので、 ソートできる。

- `sorted <https://docs.python.org/ja/3/library/functions.html#sorted>`_
- `タプル型 (tuple) <https://docs.python.org/ja/3/library/stdtypes.html#tuples>`_


all()
-----
`all(iterable) <https://docs.python.org/ja/3/library/functions.html#all>`_

- iterable の全ての要素が真ならば (もしくは iterable が空ならば) True を返す。


@property
---------
``@property`` デコレータ を付けると、プロパティのように呼び出せる。

- 付け方

  .. code-block:: python

    @ property
    def access_datehour(self):
        return self.access_datetime.strftime('%Y/%m/%d %H')

- 呼び出すとき

  .. code-block:: python

    xxx.access_datehour


リファレンス
^^^^^^^^^^^^
https://docs.python.org/ja/3/library/functions.html#property

- 同じ名前のまま 読み出し専用属性の ``getter`` にしてくれる


シーケンスのアンパッキング
--------------------------
`タプルとシーケンス <https://docs.python.org/ja/3/tutorial/datastructures.html#tuples-and-sequences>`_


StringIO().seek(0)
------------------
https://docs.python.org/ja/3/library/io.html#io.IOBase.seek

- 先頭にもどす、 (カーソルを先頭に戻すみたいなイメージ)


if __name__ == "__main__"
-------------------------
http://blog.pyq.jp/entry/Python_kaiketsu_180207

- Pythonでは、インポートされたファイルの中身は実行される


unicode と str
--------------

.. code-block:: python

  >>> # -*- coding: utf-8 -*-
  >>> 'ふみ' == u'ふみ'
  False
  >>> 'fumi23' == u'fumi23'
  True
  >>>

- python2 の場合、マルチバイトを含むと ``u`` の有無で違うオブジェクトとして判定される。
- python2の文字には ``unicode`` と ``str`` がある。 ascii 文字しか含まない場合は 同じ値と判断されるけど基本的に別物として考えたほうがいい。


リファレンス
^^^^^^^^^^^^
`3.1.3. Unicode 文字列 <https://docs.python.org/ja/2.7/tutorial/introduction.html#unicode-strings>`_


バックスラッシュ感染症
----------------------
こんなふうに書く

.. code-block:: python

  r"ab*"


リファレンス
^^^^^^^^^^^^
`バックスラッシュ感染症 <https://docs.python.org/ja/3.7/howto/regex.html#the-backslash-plague>`_

- ``r`` を文字列リテラルの先頭に書くことでバックスラッシュは特別扱いされなくなる
- 多くの場合 Python コードの中の正規表現はこの raw string 記法を使って書かれる


正規表現のグループ化機能
------------------------
このあたりから

- `取り出さないグループと名前つきグループ <https://docs.python.org/ja/3.7/howto/regex.html#non-capturing-and-named-groups>`_


長い正規表現を記述する方法
--------------------------
- カンマ区切り無しで文字列リテラルを複数に分ける

  - http://docs.python-guide.org/en/latest/writing/style/#line-continuations


- re.VERBOSE オプションを使う

  - https://docs.python.org/ja/3/library/re.html#re.VERBOSE
