.. article::
   :date: 2019-04-30
   :title: MySQL のメモ
   :category: mysql
   :tags:
   :canonical_url: /mysql/note/
   :draft: false


================
MySQL のメモ
================



リファレンス
============

- `MySQL 8.0 Reference Manual <https://dev.mysql.com/doc/refman/8.0/en/>`_
- `Chapter 25 INFORMATION_SCHEMA Tables <https://dev.mysql.com/doc/refman/8.0/en/information-schema.html>`_
- `4.5.4 mysqldump <https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html>`_
- `12.21 Window Functions <https://dev.mysql.com/doc/refman/8.0/en/window-functions.html>`_


ログイン
========

.. code-block:: bash

  # ログイン
  $ mysql -u user_name -D db_mame -p


便利
====

.. code-block:: mysql

  -- 拡張表示
  SELECT * FROM users WHERE login = 'fumi23'\G


データベース
============

.. code-block:: mysql

  -- データベース一覧
  SHOW DATABASES;

  -- データベースの切り替え
  USE db_mame;

.. code-block:: mysql

  -- 現在の Character Sets 設定を表示する
  SHOW VARIABLES LIKE "char%";

  -- 現在のタイムゾーン設定を表示する
  SHOW VARIABLES LIKE '%time_zone%';

  -- 現在の状態を表示する
  STATUS;

.. code-block:: mysql

  -- データサイズを確認する
  -- https://dev.mysql.com/doc/refman/8.0/en/tables-table.html
  SELECT
    SUM(data_length) / 1024 / 1024 / 1024 AS db_size_gb,
    SUM(data_length) / 1024 / 1024 AS db_size_mb,
    SUM(data_length) / 1024 AS db_size_kb
  FROM
    information_schema.tables
  WHERE
    table_schema = 'mmm'
  ;


テーブル
========

.. code-block:: mysql

  -- テーブル一覧
  SHOW tables;

  -- テーブルの列一覧
  SHOW COLUMNS FROM table_name;

  -- テーブル定義を確認する
  DESC table_name;
  SHOW FULL COLUMNS FROM table_name;
  SHOW CREATE TABLE table_name;


dump
====

.. code-block:: bash

  # dump を作る
  $ mysqldump -u root -p db_mame > dump_filename.sql

  # dump を入れる
  $ mysql -h localhost -u root -p db_mame < dump_filename.sql


おぼえがき
==========
- Window 関数は 8.0.2 から利用可能

  - `MySQL 8.0.2: Introducing Window Functions <https://mysqlserverteam.com/mysql-8-0-2-introducing-window-functions/>`_
