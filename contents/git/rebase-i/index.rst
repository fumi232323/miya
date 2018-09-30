.. article::
   :date: 2018-09-30
   :title: git rebase -i
   :category: git
   :tags:
   :canonical_url: /git/rebase-i/
   :draft: false

==========================
git rebase -i
==========================


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


参考にしたサイト
------------------
- これがわかりやすかった

  https://www.karakaram.com/git-rebase-i-usage#whats-rebase-i
