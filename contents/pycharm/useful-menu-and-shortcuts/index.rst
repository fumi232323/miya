.. article::
   :date: 2018-09-23
   :title: PyCharm ショートカットと便利メニュー
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/useful-menu-and-shortcuts/
   :draft: false


==========================================
PyCharm ショートカットと便利メニュー
==========================================


ガイド
======
- `PyCharm ヘルプ <https://pleiades.io/help/pycharm/>`_
- `PyCharm Help <https://www.jetbrains.com/help/pycharm/meet-pycharm.html>`_


ショートカット
=================================
- Find in Path: ``Cmd + Shift + F``
- ファイル名でファイルを探す: ``Cmd + Shift + O``
- Preferences (設定/環境設定) ダイアログを開く: ``Cmd + ,``
- 定義ジャンプ: ``Cmd + b``
- 定義ジャンプから戻る: ``Cmd + [``
- 利用箇所を検索:

  -  関数の定義（defの行）にカーソル合わせて ``Find Usages``
  - ``Option + fn + F7``

- もっと良いやり方を教えてくれる: ``Option + Enter``
- Search Everywhere (文字列検索？) : ``Shift Shift``
- アクションを探す: ``Cmd + Shift + A``
- 行コメント: ``Cmd + /``
- ブロックコメント: ``Cmd + Option + /``


便利メニュー
=================================

GitHub
-------
- GitHub で開く: コード中の開きたい箇所で右クリック -> Open on GitHub


Git
-----

merge
^^^^^^

1. acceptance ブランチをチェックアウトした状態で、
2. VCS -> Git -> Merge Changes
3. merge したいブランチに ``ON``
4. コミットメッセージ は、(書かないと) デフォルトで `Merge branch 'topic-branch-name' into acceptance` になる。
5. (コンフリクトしたら) ``Merge`` ボタン押下
6. エディターが開くので、conflicts を解消する
7. ``Apply`` ボタン押下
8. ``add`` と ``commit`` は自分でやる

    .. code-block:: console

      $ git add .
      $ git commit -m "Merge branch 'topic-branch-name' into acceptance"


rebase
^^^^^^^^

1. rebase したいブランチをチェックアウトした状態で、
2. VCS -> Git -> Rebase

    - Onto: ``master``
    - From: ``空欄``
    - ``Rebase`` ボタン

3. ``Start Rebasing`` ボタン押下
4. (コンフリクトしたら) ``Merge`` ボタン押下
5. エディターが開くので、conflicts を解消する
6. ``Apply`` ボタン押下
7. Additional Rebase Input ダイアログが開く

    -  コミットコメントは変更できる
    - ``Resume Rebasing`` ボタン押下 ( ``git rebase --continue`` 的なやつなんだろう)

8. rebase 成功したら、画面右下に `Rebase Successful` ポップアップが出現する。


stash
^^^^^^^^

- Stash する: VCS -> Git -> Stash Changes
- UnStash する: VCS -> Git -> UnStash Changes

  - stash の一覧が見られる
  - どんな stash だったか view できる
  - とても使い勝手がよい



