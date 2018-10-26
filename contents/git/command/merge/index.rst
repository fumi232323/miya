.. article::
   :date: 2018-10-26
   :title: git merge
   :category: git
   :tags:
   :canonical_url: /git/command/merge/
   :draft: false

==========================
git merge
==========================

マージする
==========
topic ブランチを acceptance ブランチへマージする。

1. acceptance にチェックアウトする

    .. code-block:: console

      $ git checkout acceptance

2. topic をマージします

    .. code-block:: console

      $ git merge topic

3. コンフリクトしたら、解消して、

    .. code-block:: console

      $ git add .
      $ git commit -m "Merge branch 'topic-branch-name' into acceptance"

4. マージに成功したら、強制push

    .. code-block:: console

      $ git push -u origin acceptance -f
