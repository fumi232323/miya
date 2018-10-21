.. article::
   :date: 2018-10-14
   :title: ssh-add
   :category: ssh
   :tags:
   :canonical_url: /ssh/command/ssh-add/
   :draft: false


==========
ssh-add
==========


リファレンス
=============
ubuntu のやつ

- `ssh-add <http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh-add.1.html>`_
- `ssh-agent <http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh-agent.1.html>`_

種類やバージョンが選べるやつ

- `SSH-ADD(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh-add&apropos=0&sektion=0&manpath=CentOS+6.5&arch=default&format=html>`_


ssh-agent
=========
公開鍵認証で使われる認証鍵を保持する

- sshで使う認証鍵を複数登録しておける
- 登録するのは秘密鍵
- 公開鍵はSSHAgentには登録できないし、登録する性質のものではない。
- 鍵に対してさらに鍵(パスフレーズ)を掛けることができるが、ssh接続するたびにパスフレーズを聞かれる。 agent に登録しておくと、つど聞かれずに済む。


ssh-add の使い方
----------------
- ssh-agent に秘密鍵を登録する

  .. code-block:: console

    $ ssh-add ~/.ssh/id_rsa
    $ ssh-add <秘密鍵のパス>


- ssh-agent に登録されている鍵を一覧表示する

  .. code-block:: console

    $ ssh-add -l


- ssh-agent に登録されている鍵をすべて削除する

  .. code-block:: console

    $ ssh-add -D


- ssh-agent 登録されている鍵から、指定した鍵を削除する

  .. code-block:: console

    $ ssh-add -d <削除したい秘密鍵のパス>


- ssh-agent のキーチェーンに秘密鍵を登録する

  .. code-block:: console

    $ ssh-add -K <秘密鍵のパス>


  - ``-K`` をつけると電源を落としても消えない。
  - 利便性は上がるが、セキュリティ度は下がる。
  - ``-K`` をつけないと、電源を落としたら消えるので、セキュリティ的に良い。
