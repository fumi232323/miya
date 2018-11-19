.. article::
   :date: 2018-09-30
   :title: git rebase
   :category: git
   :tags:
   :canonical_url: /git/command/rebase/
   :draft: false

==========
git rebase
==========


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リベース
========

master ブランチにリベースする
-----------------------------

.. code-block:: console

  $ git rebase master

コンフリクトしたらコンフリクトを解消して、

.. code-block:: console

  $ git add .
  $ git rebase --continue

リベースが終わったらフォースプッシュ

.. code-block:: console

  $ git push -f


間違えてリベースして元に戻したい
---------------------------------

push しちゃった場合
^^^^^^^^^^^^^^^^^^^

  .. code-block:: console

    $ git reflog
    $ git reset --hard HEAD@{6}  # 戻りたい番号そのものを書けばよいみたい
    $ git push -f


push してない場合
^^^^^^^^^^^^^^^^^

  間違えてリベースしたブランチを origin に戻す

    .. code-block:: console

      $ git checkout -B <branch-you-want-to-restore> origin/<branch-you-want-to-restore>


preserve-merges オプション
--------------------------
分岐してマージしたよ、という履歴みたいなのを保ったまま、リベースしてくれる

.. code-block:: console

  $ git rebase --preserve-merges master


リファレンス
^^^^^^^^^^^^
- https://git-scm.com/docs/git-rebase
- `3.6 Git のブランチ機能 - リベース <https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%83%96%E3%83%A9%E3%83%B3%E3%83%81%E6%A9%9F%E8%83%BD-%E3%83%AA%E3%83%99%E3%83%BC%E3%82%B9>`_
