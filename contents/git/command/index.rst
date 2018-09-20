.. article::
   :date: 2018-09-10
   :title: Git のコマンドまとめ
   :category: git
   :tags:
   :canonical_url: /git/command/
   :draft: false

==========================
Git のコマンドまとめ
==========================


Git の絵
===============

.. figure :: areas.png

  https://git-scm.com/book/en/v2/Getting-Started-Git-Basics


ブランチ操作
===============

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


- リモートブランチをチェックアウト

  .. code-block:: console

    $ git checkout -b <branch-you-want-to-checkout> origin/<branch-you-want-to-checkout>


- リモートブランチの一覧を表示

  .. code-block:: console

    $ git branch -a


リベース
===========

- master ブランチにリベースする

  .. code-block:: console

    $ git rebase master

    # コンフリクトしたらコンフリクトを解消して、
    $ git add .
    $ git rebase --continue

    # リベースが終わったらフォースプッシュ
    $ git push -f


- 間違えてリベースして元に戻したい

  - push しちゃった場合

    .. code-block:: console

      $ git reflog
      $ git reset --hard HEAD@{6}  # 戻りたい番号そのものを書けばよいみたい
      $ git push -f


  - push してない場合

    .. code-block:: console

      # 間違えてリベースしたブランチを origin に戻す
      $ git checkout -B <branch-you-want-to-restore> origin/<branch-you-want-to-restore>


マージする
===================

- topic ブランチを acceptance ブランチへマージする

  .. code-block:: console

    $ git checkout acceptance
    $ git merge topic
    # コンフリクトしたら、解消して、
    $ git add .
    $ git commit -m "Merge branch 'topic-branch-name' into acceptance"
    # マージに成功したら、強制push
    $ git push -u origin acceptance -f


git reset 使い分け
=========================
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
===============

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
=============================================

- これがわかりやすい

  https://qiita.com/chihiro/items/f373873d5c2dfbd03250

git log
==============

- 各コミットを 1 行ずつ表示

  .. code-block:: console

    $ git log --oneline


- master と topic の共通の祖先がわかる

  .. code-block:: console

    $ git log -1 $(git merge-base origin/master origin/topic)


- 見本がたくさん書いてあって良い

  http://yanor.net/wiki/?Git%2Fgit%20log%2F%E6%9D%A1%E4%BB%B6%E6%8C%87%E5%AE%9A%E3%81%97%E3%81%A6%E3%82%B3%E3%83%9F%E3%83%83%E3%83%88%E3%82%92%E7%B5%9E%E3%82%8A%E8%BE%BC%E3%82%80


rebase -i
===========

squash : 離れたふたつのコミットをまとめる
-------------------------------------------------------

1. くっつけ先のコミット番号を確かめる

    .. code-block:: console

      $ git log --oneline

2. コミットが一覧表示されるので、くっつけ先のコミット番号を控え、 ``:q`` で閉じる
3. rebase を開始する。

    .. code-block:: console

      $ git rebase -i <くっつけ先のコミット番号>

4. エディターが開き、 3 で指定したコミット番号より後のコミット一覧が表示されるので、くっつけたいほうの pick を一番上に移動する。
5. ``pick`` を ``squash`` に書き換える。
6. ``:wq`` で保存して閉じる
7. コミットメッセージを書く用？のエディタが開くので、

    ::

      You are currently rebasing branch 'master' on 'リビジョン番号'.

    の次の行のコメントを外して、コミットメッセージを書く。

    (たぶん、その上のほうにあるコミットメッセージも変えたければ編集したほうがよいんだと思う)

8. ``:wq`` で保存して閉じる
9. リモートリポジトリにフォースプッシュする。

    .. code-block:: console

      $ git push -f


reword: コミットメッセージを変更する
------------------------------------

1. コミット一覧を表示し、コミットメッセージを変更したいコミットの一つ前のコミット番号を控える

    .. code-block:: console

      $ git log --oneline

2. ``:q`` で閉じる
3. rebase を開始する。

    .. code-block:: console

      $ git rebase -i <1 で控えたコミット番号>

4. エディターが開き、 3 で指定したコミット番号より後のコミット一覧が表示されるので、コミットメッセージを変更したいコミット番号の ``pick`` を ``reword`` 書き換える。
5. ``:wq`` で保存して閉じる
6. コミットメッセージを書く用？のエディタが開くので、コミットメッセージを変更する。
7. ``:wq`` で保存して閉じる
8. リモートリポジトリにフォースプッシュする。

    .. code-block:: console

      $ git push -f
