.. article::
   :date: 2018-09-02
   :title: Vagrant のメモ
   :category: vagrant
   :tags:
   :canonical_url: /vagrant/note/
   :draft: false


=================
Vagrant のメモ
=================


synced_folder が効かなくなった
=================================

現象
--------
ホスト OS 側のディレクトリ構成を変えたので、 ``_repo_file`` を書き換えて、 ``vagrant reload`` したのに、エラーになった。

エラー内容
^^^^^^^^^^^

.. code-block:: console

  FumienoMacBook-Pro:ansible fumi23$ vagrant reload
  ==> default: Attempting graceful shutdown of VM...
  ==> default: Clearing any previously set forwarded ports...
  ==> default: Pruning invalid NFS exports. Administrator privileges will be required...
  Password:
  ==> default: Clearing any previously set network interfaces...
  ==> default: Preparing network interfaces based on configuration...
      default: Adapter 1: nat
      default: Adapter 2: hostonly
  ==> default: Forwarding ports...
      default: 22 (guest) => 2222 (host) (adapter 1)
  ==> default: Running 'pre-boot' VM customizations...
  ==> default: Booting VM...
  ==> default: Waiting for machine to boot. This may take a few minutes...
      default: SSH address: 127.0.0.1:2222
      default: SSH username: vagrant
      default: SSH auth method: private key
  ==> default: Machine booted and ready!
  ==> default: Checking for guest additions in VM...
      default: The guest additions on this VM do not match the installed version of
      default: VirtualBox! In most cases this is fine, but in rare cases it can
      default: prevent things such as shared folders from working properly. If you see
      default: shared folder errors, please make sure the guest additions within the
      default: virtual machine match the version of VirtualBox you have installed on
      default: your host and reload your VM.
      default:
      default: Guest Additions Version: 4.3.8
      default: VirtualBox Version: 5.1
  ==> default: Configuring and enabling network interfaces...
      default: SSH address: 127.0.0.1:2222
      default: SSH username: vagrant
      default: SSH auth method: private key
  ==> default: Exporting NFS shared folders...
  NFS is reporting that your exports file is invalid. Vagrant does
  this check before making any changes to the file. Please correct
  the issues below and execute "vagrant reload":

  exports:2: path contains non-directory or non-existent components: /Users/fumi23/my-projects/my-project-1
  exports:2: path contains non-directory or non-existent components: /Users/fumi23/my-projects/my-project-2
  exports:2: no usable directories in export entry
  exports:2: using fallback (marked offline): /

解決策
-------
``/etc/exports`` を新しい情報に書き変えた。

- 元の /etc/exports

  .. code-block:: console

    FumienoMacBook-Pro:ansible fumi23$ cat /etc/exports.bk
    # VAGRANT-BEGIN: 501 e9fe3cbf-63c6-48a1-beca-2702bc2527e4
    "/Users/fumi23/my-projects/my-project-1" "/Users/fumi23/my-projects/my-project-2" 192.168.34.10 -alldirs -mapall=501:20
    # VAGRANT-END: 501 e9fe3cbf-63c6-48a1-beca-2702bc2527e4

- 今の /etc/exports

  .. code-block:: console

    FumienoMacBook-Pro:ansible fumi23$ cat /etc/exports
    # VAGRANT-BEGIN: 501 e9fe3cbf-63c6-48a1-beca-2702bc2527e4
    "/Users/fumi23/new-my-projects/my-project-1" "/Users/fumi23/new-my-projects/my-project-2" 192.168.34.10 -alldirs -mapall=501:20
    # VAGRANT-END: 501 e9fe3cbf-63c6-48a1-beca-2702bc2527e4


参考
--------
助かりました。ありがとうございました。

- `VagrantのNFS設定でexportsのエラーが発生した <https://blog.kazu69.net/2014/07/16/by-vagrant-nfs-exports-error-has-occurred/>`_


ログを詳細に出す
=================

.. code-block:: shell

  $ VAGRANT_LOG=DEBUG vagrant [command]


VirtualBox の仮想マシンの保存先を変更する
========================================================

- `Vagrant BOXとVMの保存先を変更する方法 <http://kiraba.jp/change-save-point-vagrant-box-and-virtual-machine/>`_

  ::

    環境設定 > 一般 にあるデフォルトの仮想マシンフォルダーを任意のパスに変更すればVMは指定したフォルダーに保存されます。



VAGRANT_HOME
========================================================

環境変数: ``VAGRANT_HOME`` を設定すれば、 ``~/.vagrant.d`` の場所を好きなところに変えられそう

- `VAGRANT_HOME <https://www.vagrantup.com/docs/other/environmental-variables.html#vagrant_home>`_


ハイフンふたつの後は、普通にSSHのオプションを指定できる。
======================================================================

.. code-block:: shell

  $ vagrant ssh -- -A


VMとboxは違う
========================================================

::

  vagrant destroy で消えるのは VM 自体、 vagrant box というのは VM 作成の素になるものです。
  VM がインスタンスだとすると、 box はクラス的な。


調べる
========
- Vagrant Userの鍵認証のところがわからない


TODO
========
- paper から移動する
