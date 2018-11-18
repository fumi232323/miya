# ubuntu:16.04 イメージをベースにイメージをビルドする
FROM ubuntu:16.04

# apt で cron をインストールする
RUN apt update
RUN apt install -y cron

# task.sh と cron-example を追加する
COPY task.sh /usr/local/bin/
COPY cron-example /etc/cron.d/
# cron-example に 644 パーミッションを設定する
RUN chmod 0644 /etc/cron.d/cron-example

# cron を実行
CMD ["cron", "-f"]
