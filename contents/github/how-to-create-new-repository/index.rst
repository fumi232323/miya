.. article::
   :date: 2018-09-10
   :title: GitHub に新しいリポジトリを作成して、pushする
   :category: github
   :tags:
   :canonical_url: /gitHub/how-to-create-new-repository/
   :draft: false


======================================================
GitHub に新しいリポジトリを作成して、pushする
======================================================


GitHub に新しいリポジトリを作成して、pushする
==========================================================

1. GitHub の右上の ``+`` ボタンから、 ``New repository``
2. リポジトリ名を入力して、作成
3. こうするといいよ、というページが表示されるので、それに従う


    .. code-block:: console

      $ git remote add origin git@github.com:fumi232323/miya.git
      $ git push -u origin master
