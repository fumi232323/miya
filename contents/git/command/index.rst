.. article::
   :date: 2018-09-10
   :title: Git のコマンドまとめ
   :category: git
   :tags:
   :canonical_url: /git/command/
   :draft: false

====================
Git のコマンドまとめ
====================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


Git の絵
========

.. figure :: areas.png

  https://git-scm.com/book/en/v2/Getting-Started-Git-Basics


ブランチ操作
============

ローカル
---------

- 新しいブランチを作成して、新しいブランチにチェックアウト

  .. code-block:: console

    $ git checkout <元にしたいブランチ>
    $ git checkout -b <new-branch-name>


- ブランチ名を変更

  .. code-block:: console

    $ git branch -m <old-branch-name> <new-branch-name>


- ブランチを削除

  .. code-block:: console

    $ git checkout <違うブランチ>
    $ git branch -D <branch-you-want-to-delete>


- ブランチを強制チェックアウト（ローカル変更はなくなる）

  .. code-block:: console

    $ git checkout --force <branch-you-want-to-checkout>


- ブランチをプッシュ（初回）

  .. code-block:: console

    $ git push -u origin <branch-you-want-to-push>


リモート
---------
- リモートブランチをチェックアウト

  .. code-block:: bash

    $ git checkout -b <branch-you-want-to-checkout> origin/<branch-you-want-to-checkout>


- リモートブランチの一覧を表示

  .. code-block:: console

    $ git branch -a


git reset 使い分け
==================
ちなみに ``git reset`` == ``git reset --mixed`` ですね


- git commit だけ取り消し

  .. code-block:: console

    $ git reset --soft


- git add と git commit を取り消し、ファイルの変更は保持する

  .. code-block:: console

      $ git reset --mixed


- git add と git commit を取り消して、ファイルの変更も削除する

  .. code-block:: console

      $ git reset --hard


- コミットを取り消し（直前のコミットまで戻す。 git commit を取り消し、ファイルの変更は保持する。）

  .. code-block:: console

    $ git reset --mixed HEAD^


コミット
========

- コミットをもう一度やりなおす

  .. code-block:: console

    $ git commit --amend


- コミットメッセージの修正

  .. code-block:: console

    $ git commit --amend -m "new commit message"


- いったんコミットした後、 add 忘れに気づいた

  .. code-block:: console

      $ git add <わすれもの>
      $ git commit --amend


stash: コミットはせずに変更を退避したい
=======================================

- これがわかりやすい

  https://qiita.com/chihiro/items/f373873d5c2dfbd03250


git log
=======

- 各コミットを 1 行ずつ表示

  .. code-block:: console

    $ git log --oneline


- master と topic の共通の祖先がわかる

  .. code-block:: console

    $ git log -1 $(git merge-base origin/master origin/topic)


- 見本がたくさん書いてあって良い

  http://yanor.net/wiki/?Git%2Fgit%20log%2F%E6%9D%A1%E4%BB%B6%E6%8C%87%E5%AE%9A%E3%81%97%E3%81%A6%E3%82%B3%E3%83%9F%E3%83%83%E3%83%88%E3%82%92%E7%B5%9E%E3%82%8A%E8%BE%BC%E3%82%80


リセットする
============
履歴を全部消して force push する。

1. ``.git`` を消す
2. force push する

    .. code-block:: bash

      $ git add *
      $ git commit -m 'initialize
      $ git remote add origin {URL}
      $ git push origin master --force


fetch と pull
=============

fetch
------
リモートリポジトリの最新の履歴の取得だけを行う。

- ``hogehoge`` ブランチをfetchすると、 ローカルの ``origin/hogehoge`` がリモートの ``origin/hogehoge`` リポジトリと同期されて最新状態になる。
- ローカルの ``hogehoge`` ブランチは、そのまま何にも変更されない。

pull
-----
fetch + merge

- ``hogehoge`` ブランチを pull すると、 ローカルの ``origin/hogehoge`` も ``hogehoge`` も両方リモートの ``origin/hogehoge`` リポジトリの変更とマージされる。
- 内部的には、

  1. リモートの ``origin/hogehoge`` と、ローカルの ``origin/hogehoge`` とマージ
  2. ローカルの ``origin/hogehoge`` と ``hogehoge`` をマージ

- ローカルの ``hogehoge`` に、自分の変更とリモートの変更と両方入った状態になる。
- 競合があったら自分で解決してコミットする必要がある。
- 通常ローカルで触るのは ``origin`` がついていない ``hogehoge`` ブランチ。
