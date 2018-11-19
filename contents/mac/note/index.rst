.. article::
   :date: 2018-11-19
   :title: Mac のメモ
   :category: mac
   :tags:
   :canonical_url: /mac/note/
   :draft: false


==========
Mac のメモ
==========


ファイル操作
============

文字コードを変換
----------------

.. code-block:: console

  $ iconv -f Shift-JIS -t UTF-8 <変換前のファイル名> <変換後のファイル名>


.gz に圧縮
----------

.. code-block:: console

  $ gzip <ファイル名>


環境変数
========

環境変数の一覧を表示する
------------------------

.. code-block:: console

  $ printenv


環境変数を追加する
------------------

やりかたがわからない


本体がどこに行ったかわからなくなったとき
========================================

.. code-block:: console

  $ which python
