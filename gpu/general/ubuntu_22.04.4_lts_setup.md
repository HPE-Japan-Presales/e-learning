# GPUドライバーをインストールする前段階のOS設定（Ubuntu Server 22.04.4 LTS）
- OS Ubuntu Server 22.04.4 LTS
- OSインストールでは、HWEカーネルを選択


# パッケージリポジトリの更新
    # sudo su -
    # apt update


#　ツール類インストール
    # apt install -y vim iputils-ping net-tools curl wget w3m ncdu htop mc git

# 日本語ロケールの設定（ただしアプリの要件による）
    # apt install -y language-pack-ja-base language-pack-ja
    # localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja"
    # source /etc/default/locale
    # localectl status
    # date

# アプリの都合上、英語に設定する場合
    # vi /etc/default/locale
      LANG=C
      LANGUAGE=C
    # source /etc/default/locale

# タイムゾーン変更
    # timedatectl status
    # timedatectl set-timezone Asia/Tokyo
    # timedatectl status
    # date

# ファイアウォール停止
    # apt install -y ufw iptables
    # ufw disable
    # ufw status
    # iptables -L

# 時刻同期確認
    # systemctl status systemd-timesyncd

# 自動更新の停止
    # vi /etc/apt/apt.conf.d/20auto-upgrades
    APT::Periodic::Update-Package-Lists "0";
    APT::Periodic::Unattended-Upgrade "0";

# シャットダウン時のタイムアウト時間変更
    # vi /etc/systemd/system.conf
    ...
    DefaultTimeoutStopSec=10s
    ...

# ハイバネーション無効化
    # systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
    # systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target

# マルチユーザーモード設定
    # systemctl set-default multi-user

# IPフォワード設定
    # vi /etc/sysctl.conf 
    net.ipv4.ip_forward=1

    # sysctl -p
    # sysctl -a | grep .ip_forward
    net.ipv4.ip_forward = 1
    net.ipv4.ip_forward_update_priority = 1
    net.ipv4.ip_forward_use_pmtu = 0

[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))
