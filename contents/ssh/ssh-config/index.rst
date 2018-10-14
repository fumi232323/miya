.. article::
   :date: 2018-10-14
   :title: ssh config
   :category: ssh
   :tags:
   :canonical_url: /ssh/ssh-config/
   :draft: true


==========
ssh config
==========

TODO: 整理中
TODO: ssh config のメモとして整理しようと思う

リファレンス
=============
ubuntu のやつ

- `ssh_config <http://manpages.ubuntu.com/manpages/bionic/en/man5/ssh_config.5.html>`_

種類やバージョンが選べるやつ

- `SSH(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh&apropos=0&sektion=0&manpath=CentOS+6.5&arch=default&format=html>`_


~/.ssh/config
-------------
ユーザごとの個人用設定ファイルです。ファイル形式は上で説明されています。このファイルは SSH クライアントによって使われます。このファイルは潜在的に悪用される危険性があるため、パーミッションは厳しくしておくのがいいでしょう。所有者のみが読み書き可能で、他からはアクセスできないようにしておくべきです。 |

/etc/ssh/ssh_config
-------------------
システム全体にわたる設定ファイルです。このファイルはユーザの設定ファイルでは指定されなかった値を提供し、また設定ファイルを持たないユーザのためのデフォルトにもなります。このファイルは誰にでも読み込み可能でなければいけません。

ProxyCommand
============

ローカルから、B へ ssh する

.. code-block:: bash

  $ ssh-add ~/.ssh/id_rsa
  $ ssh my-host

  1. まず、(1) の ``ssh -W %h:%p my-gateway`` が実行される
  2. ``$ ssh my-gateway`` が実行される
    - HostName, User, IdentityFile は (3) に記述のものが使用される
    -
  3. ``my-gateway`` から ``$ ssh my-host`` が実行される
    - ``ForwardAgent yes`` のため、 ``IdentityFile`` は、 (3) で指定されたものが使用される


``~/.ssh/config``

  .. code-block:: console

    # エージェント転送を行う
    ForwardAgent yes

    # A: gateway
    Host my-gateway                                 # (3)
      HostName      my-gateway.com
      User          my-user                         # (2)
      IdentityFile  ~/.ssh/id_rsa

    # B: アプリケーションがデプロイしてあるサーバー
    Host my-host
      HostName      my-host-name
      User          my-user                         # (2)
      ProxyCommand  ssh -W %h:%p my-gateway         # (1)


(1)

キーワードやオプションの意味
----------------------------------------------
- ssh_config のマニュアルページの ``ProxyCommand`` の説明

  ::

    Specifies the command to use to connect to the server.
    The command string extends to the end of the line, and is executed with the user's shell.
    In the command string, `%h' will be substituted by the host name to connect and `%p' by the port.
    The command can be basically anything, and should read from its standard input and write to its standard output.

- ssh のマニュアルページの ``-W`` オプションの説明

  ::

    -W host:port
      Requests that standard input and output on the client be forwarded to host on port over the secure channel.


参考
----
- `ProxyCommand <http://note.crohaco.net/2017/ssh-tunnel/#proxycommand>`_


ForwardAgent yes
================
ForwardAgent (エージェント転送)
認証エージェントへの接続を、(それが存在する時は) リモートマシン上に転送するかどうかを指定します。この引数の値は"yes"あるいは"no"でなければならず、デフォルトは"no (エージェント転送をおこなわない)"です。
認証エージェントの転送には注意が必要です。リモートホスト上で (エージェントの UNIX ドメインソケットに対する)ファイルアクセス権限を無視できてしまうユーザがいる場合は、転送された接続を介してローカル側の認証エージェントにアクセスできてしまうことになります。攻撃側は認証エージェントから鍵そのものを盗むことはできませんが、認証エージェントがもっている鍵に認証をおこなわせることはできます。
