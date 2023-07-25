# HPE Ezmeral Data Fabric 6.2 Enterprise Editionを最小構成でインストール

## はじめに
この構成は本番環境での使用では推奨されませんので注意してください。

## 構成
- Data Fabric Node x3
  - HW: VM
  - OS: [Ubuntu 18.04](https://docs.datafabric.hpe.com/62/InteropMatrix/r_os_support_matrix.html)
  - CPU: [6cores](https://docs.datafabric.hpe.com/62/AdvancedInstallation/PlanningtheCluster-hardware.html)
  - RAM: [16GB](https://docs.datafabric.hpe.com/62/AdvancedInstallation/PreparingEachNode-memory.html)
  - OS Disk: [150GB (vmdk)](https://docs.datafabric.hpe.com/62/AdvancedInstallation/PreparingEachNode-memory.html)
  - Data Disk: 100GB (vmdk)
  - Network: 10Gbps (インターネット接続可能)

## インストール手順
### 基本セットアップ
ネットワーク、NTP、DNS、ホスト名等の基本セットアップを完了してください。まずは１台のVMで設定を行い、残り2台はクローンします。

### Apparmor無効化

```bash
$ systemctl disable apparmor
```

### Kernelパラメータの設定
[こちらのドキュメント](https://docs.datafabric.hpe.com/62/Performance/tuning-system-performance.html)通りにKernelパラメータを設定します。*/etc/sysctl*に以下を追加します。

```bash
############
# For MAPR
############
fs.aio-max-nr=262144
fs.epoll.max_user_watches=32768
fs.file-max=32768
net.ipv4.route.flush=1
net.core.rmem_max=4194304
net.core.rmem_default=1048576
net.core.wmem_max=4194304
net.core.wmem_default=1048576
net.core.netdev_max_backlog=30000
net.ipv4.tcp_rmem=4096 1048576 4194304
net.ipv4.tcp_mem=8388608 8388608 8388608
net.ipv4.tcp_syn_retries=4
net.ipv4.tcp_retries2=5
vm.dirty_ratio=6
vm.dirty_background_ratio=3
vm.overcommit_memory=0
vm.swappiness=1
```

### Diskパラメータ設定
Diskパラメータを設定します。使用しているDiskは２つで、1つはOS用に使用しているためパーティションが切られています。

```bash
$ ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda3  /dev/sdb
```

今回のデータディスクは*/dev/sdb*なので、以下のようにパラメータを設定してください。永続化するために*/etc/rc.local*に記載しておきます。

```bash
$ vi /etc/rc.local 
#!/bin/sh
echo "1024" > /sys/block/sdb/queue/max_sectors_kb
echo noop > /sys/block/sdb/queue/scheduler

$ chmod +x /etc/rc.local 
$ reboot
```

### クローン
ここまで完成したら、VMをクローンして、IPセットアップを終えた後、Data Fabric Nodeを３台起動してください。

### インストーラー
Data Fabric Node01にrootユーザーでログインして、インストーラを取得します。

```bash
$ mkdir -p /root/work/datafabric
$ cd /root/work/datafabric
$ wget https://package.mapr.hpe.com/releases/installer/mapr-setup.sh
--2022-09-16 10:24:56--  https://package.mapr.hpe.com/releases/installer/mapr-setup.sh
Resolving package.mapr.hpe.com (package.mapr.hpe.com)... 18.65.168.113, 18.65.168.114, 18.65.168.5, ...
Connecting to package.mapr.hpe.com (package.mapr.hpe.com)|18.65.168.113|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 149507 (146K) [text/x-sh]
Saving to: ‘mapr-setup.sh’

mapr-setup.sh             100%[=====================================>] 146.00K   172KB/s    in 0.8s    

2022-09-16 10:24:58 (172 KB/s) - ‘mapr-setup.sh’ saved [149507/149507]

$ chmod +x mapr-setup.sh 
```

インストーラーを起動させます。

```bash
$ ./mapr-setup.sh 
                                                    
                     HPE Ezmeral Data Fabric Distribution Initialization and Update
                                                    
             Copyright 2022 Hewlett Packard Enterprise Development LP., All Rights Reserved
                          https://www.hpe.com/us/en/software/data-fabric.html

WARNING: unix_chkpwd does not have the suid bit set so local password authentication will fail 

Continue install anyway? (y/n) [y]: y
Install required packages? (y/n) [y]: y

Installing installer package dependencies... 
```

あとはインストーラーの指示に従って、値を入力してください。最終的にはWeb Consoleが起動します。

```
...Success 

            To continue installing HPE Ezmeral Data Fabric software, open the following URL in a web browser
                                                            
                                      https://datafabric01b.hybrid-lab.local:9443
```

*mapr*ユーザーでログインします
![](pics/web_installer01.png)

![](pics/web_installer02.png)

今回は6.2.0を使いたいので、そのバージョンを選択する。
![](pics/web_installer03.png)

最小構成なので余計なセキュリティ機能は無効にします。NFSは有効にして、あとでNFSでファイルをアップロードしてみましょう。
![](pics/web_installer04.png)

MapR Ecosystem Packはhttpfsだけ有効にします。あとでhttpfsを使ってファイルが参照できるか確認してみます。
![](pics/web_installer05.png)


![](pics/web_installer06.png)

最小構成なのでGrafanaを無効にしときます。
![](pics/web_installer07.png)

maprユーザーのパスワードを変更します。
![](pics/web_installer08.png)

クラスタを組むノードを全てを加えます。diskは*/dev/sdb*の100GB Diskです。
![](pics/web_installer09.png)

プリチェック結果を待ちます。
![](pics/web_installer10.png)

クローンしたDataFabric Node2-3のIPを間違えたので、エラーになりました:(  
修正して、再度プリチェックを流します。
![](pics/web_installer10-1.png)

JDKがないよーと言われていますが、自動でインストールされることを期待して次に進みます。
![](pics/web_installer10-2.png)

ステータスも100%になっていることは忘れずに確認してください。
![](pics/web_installer11.png)

サマリーを確認してinstallボタンを押します。
![](pics/web_installer12.png)

「3nodeは本番環境で使えないけど大丈夫?」というメッセージが出てきました。無視して進めます。
![](pics/web_installer13.png)

インストールが実行されますので待ちます。
![](pics/web_installer14.png)

実はこの手順で１度インストールが失敗しました。*/opt/mapr/logs/configure.log*に以下のようなメッセージが書き込まれていました。

```bash
2022-09-16 11:07:52.709 datafabric01 configure.sh(30162) Install GenerateSslKeys:1459 ERROR: could not generate ssl keys

```

OpenSSLもインストールされているみたいなので、GUIに従って再度インストールしたら無事にインストール完了しました。

![](pics/web_installer15.png)

今回はEnterprise Editionを選択したのでライセンスについての注意事項を読みます。

![](pics/web_installer16.png)

インストールが完了しました。"Click here to go to HPE Ezmeral Data Fabric Control System"を押して、GUI管理画面に移動します。

![](pics/web_installer17.png)

### ステータス確認
MCSに*mapr*ユーザーでログインします。

![](pics/mcs01.png)

ELUA的なのが表示されますので、Agreeします。

![](pics/mcs02.png)

CLDBとNFSのサービスがDegradeしていますが、しばらく待ってステータスが戻るか確認してみます。戻らないようでしたらノード再起動します。

![](pics/mcs03.png)

NFSサービスが全然正常にならないのでログを確認します。
![](pics/mcs03-1.png)

```bash
$ cat /opt/mapr/logs/nfsserver.log
2022-09-16 12:00:18,3272 INFO nfs:13747 fs/nfsd/nfsha.cc:1057 exiting: No license to run NFS server in servermode
```

ライセンスがないようです。。。NFRライセンスを取得して適用してみます。Admin > Cluster setting > LicenseでCluster IDをメモします。ライセンス発行に際に必要です。
![](pics/mcs04.png)


ライセンスを取得したらAdmin > Cluster setting > Licenseに進んで適用します。
![](pics/mcs05.png)

ServiceからNFS V3 Gatewayを起動させてみます。
![](pics/mcs06.png)

無事NFSが正常になりました。
![](pics/mcs07.png)


### 動作確認
Data>Volume>Create VolumeからVolumeを作成します。
![](pics/volume01.png)

*test01*というボリュームを作ってみます。
![](pics/volume02.png)

Data Fabric Node01でマウントしてみます。

```bash
$ cd /tmp
$ mkdir mapr_nfs
$ mount -t nfs  <Data Fabric Node02名>:/mapr/<Data Fabricクラスタ名>/test01 ./mapr_nfs/
#i.e. mount -t nfs  datafabric02b:/mapr/datafabric.hybrid-lab.local/test01 ./mapr_nfs/

$ echo "Hello my Data Fabric" > ./mapr_nfs/test01
```

次に端末からtest01をマウントしています。使用していつ端末はMACです。

```bash
MAC$ sudo mount -o nolocks -t nfs datafabric02b.hybrid-lab.local:/mapr/datafabric.hybrid-lab.local/test01  ./mapr_nfs/ 
MAC$  /tmp cat mapr_nfs/test01            
Hello my Data Fabric
```

無事に小さい構成でもNFSでDataFabric内のファイルが確認できました。冒頭でも言っていますが、あくまで検証構成になることをご注意ください。