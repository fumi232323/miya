.. article::
   :date: 2018-08-05
   :title: ProxyCommand とは何ですか
   :category: ssh
   :tags:
   :canonical_url: /ssh/ssh-config-proxy-command/
   :draft: false


ある日
=========================
ssh-config を眺めていたら、こうなっていました。

- ~/.ssh/config

  .. code-block:: bash

    ForwardAgent yes
    ServerAliveInterval 300

    # ホスト A
    Host my-gateway
      HostName      my.gateway.com
      User          myuser
      IdentityFile  ~/.ssh/id_rsa

    # ホスト B
    Host myhost
      HostName      myhost-name
      User          myuser
      ProxyCommand  ssh -W %h:%p my-gateway  # <- ここが不思議


今まで、とにかく無事につながってくれさえすれば何でもよいと思ってきましたが、これは何ですか・・・

こういうことでした
=========================
ローカルから、ホスト B へ ssh する際に、

.. code-block:: bash

  $ ssh myhost

としていたのですが、間にもっとたくさんはさまっていました。

間にはさまっていたこと
-----------------------
1. ホスト B へ行くのに、
2. ホスト A に `IdentityFile` で ssh 接続したあと、
3. ホスト A から、ホスト B に ssh 接続する、
4. `ForwardAgent yes` しているから、鍵はそのまま転送ね。


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


参考にしたページ
===================
- `SSH_CONFIG(5) <https://www.freebsd.org/cgi/man.cgi?query=ssh_config&apropos=0&sektion=0&manpath=CentOS+6.5&arch=default&format=html>`_
- `SSH(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh&apropos=0&sektion=0&manpath=CentOS+6.5&arch=default&format=html>`_
- `ProxyCommand <http://note.crohaco.net/2017/ssh-tunnel/#proxycommand>`_


ありがとうございました。
