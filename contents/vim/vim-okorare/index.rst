.. article::
   :date: 2018-07-30
   :title: vim に怒られていたのでなおした
   :category: vim
   :tags:
   :canonical_url: /vim/vim-okorare/
   :draft: false


どういうわけか
=========================
- 何にもしていないのに（たぶん何かしたんだと思う）、 vim が怒って起動しなくなったので、なおすことにした。

- こう言っていた。

  .. code-block:: shell

    Error detected while processing /Users/fumi23/.vimrc:
    line   19:
    E484: Can't open file /usr/share/vim/vim74/defaults.vim
    line   60:
    E492: Not an editor command:   packadd! matchit
    Press ENTER or type command to continue

- 実際に ``/usr/share/vim/vim74/defaults.vim`` あるかな？と確認したら、ちゃんとなかった。消してないけど・・（消したのかもしれない）？不思議。

- ``defaults.vim`` って何？？

  - こういうことだそうです↓

    - https://qiita.com/thinca/items/9a42ef9047d44a765bdd#%E8%83%8C%E6%99%AF

      ::

        Vim には今でも多くの新機能が追加され便利になっていっていますが、一方で互換性も重視しています。
        中には、シンタックスハイライトなど明らかに便利であるにも関わらず、デフォルトでは有効になっていない機能があります。
        新しく Vim を使い始めるユーザーにとって、互換性が理由で便利な機能がすぐに使えないことはあまり嬉しくはないでしょう。
        そこで追加されたのが defaults.vim です。あくまで Vim 自体のデフォルト値は変えずに、ユーザーに便利な設定を提供します。


どうしたか
=========================
再インストールしたら失くしたものを取り戻せるのでは？？と考え、 アンインストール -> インストール することにした。

1. vim をアンインストール

    .. code-block:: shell

      $ brew uninstall vim
      Uninstalling /usr/local/Cellar/vim/8.1.0202... (1,434 files, 23.4MB)


    インストール時に ``brew`` でやったような覚えがあった


2. アンインストールできたか確認してみる。

    .. code-block:: shell

      $ which vim
      /usr/bin/vim

    あれ・・ある。 mac にデフォルトで入っていたやつかな・・・

    .. code-block:: shell

      $ vim --version
      -bash: /usr/local/bin/vim: No such file or directory

    今度はないって言う。でもよく見たら↑とパスが違うな・・ ``/usr/local/bin/vim``

    これならどうかな

    .. code-block:: shell

      $ /usr/bin/vim --version
      VIM - Vi IMproved 7.4 (2013 Aug 10, compiled Apr  4 2017 18:14:54)
      Included patches: 1-898, 8056
      (以下省略)

    なんか出た。

    どうして、ふたつもみっつも（みっつはないかもしれない）あるんだろう・・・

    これもアンインストールしたいと思ってごにょごにょ調べた結果、

      - 手動でひとつずつファイルを削除するっぽい。（めんどう）

      - 古いバージョンのものがあっても、手動もしくは自動で置き換えられるっぽい。

3. というわけで、無視して再度インストールしてみる。

    .. code-block:: shell

      $ brew install vim
      (以下省略)

    けっこう長かった。

4. インストールできたか確認してみる。

    .. code-block:: shell

      $ which vim
      /usr/local/bin/vim

    む。さっき見たパスだ・・・ ``/usr/local/bin/vim``

    .. code-block:: shell

      $ vim --version
      VIM - Vi IMproved 8.1 (2018 May 18, compiled Jul 22 2018 05:24:59)
      macOS version
      Included patches: 1-202
      (以下省略)


5. 適当なファイルを開いてみる。

    なおった。

参考にしたサイト
=========================
- https://qiita.com/thinca/items/9a42ef9047d44a765bdd
- https://oki2a24.com/2016/08/17/install-uninstall-start-vim-with-homebrew-in-mac/

ありがとうございました。
