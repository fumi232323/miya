.. article::
   :date: 2018-09-25
   :title: PyCharm で SSH tunnel して Amazon Redshift につなぐ
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/connect-db-using-ssh-tunnel/
   :draft: false


=======================================================
PyCharm で SSH tunnel して Amazon Redshift につなぐ
=======================================================


ガイド
======
- `PyCharm ヘルプ <https://pleiades.io/help/pycharm/>`_


手順
============
1. View -> Tool Windows -> Database で、スパナのボタン押下
2. Data Sources and Drivers ダイアログで、 ``+`` ボタン押下
3. ``Amazon Redshift`` を選択
4. General タブ

    - Name: 任意の名前
    - Comment: 任意のコメント
    - User: 接続先 DB のユーザー名
    - Password: 接続先 DB のユーザーのパスワード
    - URL: AWS コンソール -> Redshift ダッシュボード -> クラスターメニュー -> 接続したいクラスター -> クラスターデータベースのプロパティ -> JDBC URL に記載の URL

      - 末尾は接続先 DB 名 ( ``port番号/`` のあと)

    - URL の右横のプルダウン: ``URL only``
    - Driver: ``Amazon Redshift``

5. SSH/SSL タブ

    - Use SSH tunnel: ``ON``
    - Proxy host: gateway (HostName)
    - Port: ``22`` (SSH接続時のPort番号)
    - Proxy user: gateway のユーザー名
    - Auth type: ``OpenSSH config and authentication agent``

6. Test Connection してみる


参考URL
============
- `データベースへの接続 -> Amazon Redshift <https://pleiades.io/help/pycharm/connecting-to-a-database.html#amazon_redshift>`_
- `データベースへの接続 -> SSHによる接続 <https://pleiades.io/help/pycharm/connecting-to-a-database.html#connect_via_ssh>`_
