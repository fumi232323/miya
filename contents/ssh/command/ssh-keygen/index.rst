.. article::
   :date: 2018-10-15
   :title: ssh-keygen
   :category: ssh
   :tags:
   :canonical_url: /ssh/command/ssh-keygen/
   :draft: false


==========
ssh-keygen
==========


リファレンス
=============
ubuntu のやつ

- `ssh-keygen <http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh-keygen.1.html>`_

種類やバージョンが選べるやつ

- `SSH-KEYGEN(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh-keygen&apropos=0&sektion=1&manpath=CentOS+6.5&arch=default&format=html>`_


ssh-keygen
==========
認証鍵の生成と管理・変換が役割


秘密鍵・公開鍵のペアを作る
--------------------------

.. code-block:: bash

  $ ssh-keygen -t rsa

- ``~/.ssh/id_rsa`` (秘密鍵), ``~/.ssh/id_rsa.pub`` (公開鍵) にできあがる。

  - ``id_rsa`` が、秘密鍵(自分で持っておく、復号用)
  - ``id_rsa.pub`` が、公開鍵(サーバに置く、暗号用)

- ``-t type`` : 生成する鍵のタイプを指定できる
- ``-f output_keyfile`` : 出力先ファイルを指定できる
- ``-N new_passphrase`` : パスフレーズ（鍵の鍵みたいなやつ）を指定できる。後からでもつけられる。
- ``-C comment`` : コメント


サーバ側の公開鍵の置き場所
^^^^^^^^^^^^^^^^^^^^^^^^^^
$HOME/.ssh/authorized_keys


秘密鍵から公開鍵を取り出す
--------------------------

.. code-block:: bash

  $ ssh-keygen -y -f private_key_file


公開鍵から fingerprint を表示
------------------------------------
fingerprint は、公開鍵にくっついているものらしい。

- カレントディレクトリ配下の公開鍵ファイルを指定して表示

  .. code-block:: bash

    $ ssh-keygen -l -f id_rsa_test1.pub
    2048 SHA256:1WfZC2LtB0bkllFW1bTmSpleRbR+QSnARuYFvT7kLC0 fumi23 (RSA)

- 秘密鍵を指定すると、ペアとなる公開鍵を探してその fingerprint を表示してくれる。

  .. code-block:: bash

    $ ssh-keygen -l -f id_rsa_test1
    2048 SHA256:1WfZC2LtB0bkllFW1bTmSpleRbR+QSnARuYFvT7kLC0 fumi23 (RSA)


known_hosts file からキーを削除
------------------------------------

.. code-block:: bash

  $ ssh-keygen -R hostname [-f known_hosts_file]


ためしてみる
------------
1. カレントディレクトリに、↓で rsa 鍵のペアを作成する。

  - ファイル名: ``id_rsa_test1``
  - コメント: ``fumi23``
  - パスフレーズ: ``fumi23``

    .. code-block:: bash

      $ ssh-keygen -t rsa -f id_rsa_test1 -C fumi23
      Generating public/private rsa key pair.
      Enter passphrase (empty for no passphrase): (ここで ``fumi23`` と入力した)
      Enter same passphrase again:  (ここで ``fumi23`` と入力した)
      Your identification has been saved in id_rsa_test1.
      Your public key has been saved in id_rsa_test1.pub.
      The key fingerprint is:
      SHA256:1WfZC2LtB0bkllFW1bTmSpleRbR+QSnARuYFvT7kLC0 fumi23
      The key's randomart image is:
      +---[RSA 2048]----+
      |          o===.*@|
      |          o+=o==+|
      |          o+.O*=o|
      |         .. ==O +|
      |        S   *= *.|
      |           Eo*+ .|
      |            oo.  |
      |                 |
      |                 |
      +----[SHA256]-----+

2. できあがった。

    カレントディレクトリに、秘密鍵ファイルと公開鍵ファイルが作成できた。

    .. code-block:: bash

      $ ls -la
      -rw-------   1 fumi23  staff   1766  8  5 16:40 id_rsa_test1
      -rw-r--r--   1 fumi23  staff    388  8  5 16:40 id_rsa_test1.pub

3. 公開鍵の中身を見てみる。

    末尾にコメントとして指定した文字列が付与されている。

    .. code-block:: bash

      $ cat id_rsa_test1.pub
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfq5BPUHIRnrxBX1b3sp8OFTzFh3k9e1VZ3OXlRQRAKPTJlwdMR0apIatgq4KocFTTc4EKBksOVxOJShG1iVcUNFkhQ0kxpHTMyPHMyQdgpWAqaF5REOKMCI111xWgEC166zLUwZ1SdOHi/p2+5oDFhElsyjprro66o+uVluCD1VmfWORYYZlrMyUTtbdzHOO8xyT4k+yVMnuDJSLgfSGkCA/gXUi9vCqJf0p5iRt1owf520DSLLnkE5Cu9QxIdGDEBbS8lq53oJm5DyOcSXn+V2vKBv6pfjh+TJJNZ6PClrRI7Zk/aZFAkB/9XgqErbhU6mkHWWO9vmRavJh8Wspd fumi23

4. 秘密鍵から公開鍵を取り出してみる。

    パスフレーズを聞かれる。 3 と同じ公開鍵が取り出せた。

    .. code-block:: bash

      $ ssh-keygen -y -f id_rsa_test1
      Enter passphrase: (ここで ``fumi23`` と入力した)
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfq5BPUHIRnrxBX1b3sp8OFTzFh3k9e1VZ3OXlRQRAKPTJlwdMR0apIatgq4KocFTTc4EKBksOVxOJShG1iVcUNFkhQ0kxpHTMyPHMyQdgpWAqaF5REOKMCI111xWgEC166zLUwZ1SdOHi/p2+5oDFhElsyjprro66o+uVluCD1VmfWORYYZlrMyUTtbdzHOO8xyT4k+yVMnuDJSLgfSGkCA/gXUi9vCqJf0p5iRt1owf520DSLLnkE5Cu9QxIdGDEBbS8lq53oJm5DyOcSXn+V2vKBv6pfjh+TJJNZ6PClrRI7Zk/aZFAkB/9XgqErbhU6mkHWWO9vmRavJh8Wspd

5. 公開鍵のほうは、RSA認証で接続したいサーバーの ``~/.ssh/authorized_keys`` に追記する。


参考
^^^^
http://note.crohaco.net/2014/public-key-basic-config/

