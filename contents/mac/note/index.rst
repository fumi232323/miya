.. article::
   :date: 2018-09-02
   :title: Mac のメモ
   :category: mac
   :tags:
   :canonical_url: /mac/note/
   :draft: false


環境変数の一覧を表示する
=====================================

.. code-block:: shell

  $ printenv


環境変数を追加する
=====================
多分これ
https://qiita.com/kzhrk/items/9668a0b1a6af47bab31b

- やってないので、あとで試す


本体がどこに行ったかわからなくなったとき
===================================================

.. code-block:: shell

  $ which python


文字コードを変換
=================

.. code-block:: shell

  $ iconv -f Shift-JIS -t UTF-8 {変換前のファイル名} > {変換後のファイル名}
