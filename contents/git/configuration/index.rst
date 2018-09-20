.. article::
   :date: 2018-09-20
   :title: Git の設定まとめ
   :category: git
   :tags:
   :canonical_url: /git/configuration/
   :draft: false

==========================
Git の設定まとめ
==========================

- エディタに Vim に設定する

  .. code-block:: console

    $ git config --global core.editor 'vim -c "set fenc=utf-8"'

- git status で日本語のファイル名をちゃんと出す

  .. code-block:: console

    $ git config --global core.quotepath off

- 現在の設定を確認する

  .. code-block:: console

    $ git config --list