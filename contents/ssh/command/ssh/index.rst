.. article::
   :date: 2018-10-14
   :title: ssh のメモ
   :category: ssh
   :tags:
   :canonical_url: /ssh/command/ssh/
   :draft: false


==========
ssh
==========


リファレンス
=============
ubuntu のやつ

- `ssh <http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh.1.html>`_
- `ssh_config <http://manpages.ubuntu.com/manpages/bionic/en/man5/ssh_config.5.html>`_

種類やバージョンが選べるやつ

- `SSH(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh&apropos=0&sektion=0&manpath=CentOS+6.5&arch=default&format=html>`_
- `SSH_CONFIG(5) <https://www.freebsd.org/cgi/man.cgi?query=ssh_config&apropos=0&sektion=0&manpath=CentOS+6.5&arch=default&format=html>`_


sshとは
=======
SecureShell

- リモートマシンにログインし、リモートマシン上でコマンドを実行できるプログラム
- セキュアでないネットワーク上の、信頼できないホスト間で、セキュアな暗号化通信を提供する


ssh の使い方
------------
- リモートホストに接続する

  .. code-block:: console

    $ ssh <hostname>


- 秘密鍵を指定して接続する

  .. code-block:: console

    $ ssh -i <identity_file> <user>@<hostname>

  - ``-i <identity_file>`` : 公開鍵認証のためのID（秘密鍵）ファイルを指定する
  - identity files は、 ssh config ファイル内でホストごとに指定することもできる


オプションいろいろ
------------------
- ``-v`` : 詳細を表示する

  - Multiple -v options increase the verbosity.  The maximum is 3.

- ``-A`` : ssh-agent 転送を有効にする

  - ssh config ファイル内でホストごとに指定することもできる

- オプションで指定できるものは (できないものも) だいたい、 ssh config ファイルでも指定できる。


ssh トンネル (Port forwardings)
================================
ローカルに対するアクセスをリモートに受け流す。

ファイアウォールの中にあって直接接続できない DB1 に、 server1 を介して接続するときに使う。

.. code-block:: console

  $ ssh -fN <踏み台ホスト> -L <ローカルのポート>:<リモートホスト>:<リモートホストのポート>


- ``-L [bind_address:]port:host:hostport`` : ローカルへの通信をリモートにバインドする
- ``-N`` : Do not execute a remote command.  This is useful for just forwarding ports.
- ``-f`` : バックグラウンドで実行してね


参考
----
http://note.crohaco.net/2017/ssh-tunnel/#local-forwarding
