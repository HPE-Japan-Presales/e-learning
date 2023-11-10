# Ezmeral Unified Analyticsを最小構成でオンプレにインストール

## 要件

- vSphere 7.0u2以上 x2台以上
- 1TB使えるくらいの早いデータストア(遅いHDDだとタイムアウトする可能性あり)
- vCenter Server
- インストーラー立ち上げ用Docker環境(AMD64系プロセッサー)
- DNS/NTPサーバー
- 適当なドメイン(今回はUnified Analytics用にhybrid-lab.localを用意)
- インターネット接続可能
- NFRライセンスを[HPE Software Center](https://myenterpriselicense.hpe.com/)から入手できること(インストール後に表示されるクラスタIDが必要なので、後で入手します)


今回は最小構成でやります。構成されるノード構成は以下です。(デプロイされるプラットフォームの実態Kubernetesです)

- master x1: vCPU 8, RAM 24GB
- worker x3: vCPU 32, RAM 128GB

上記ノードをデプロイできるHWをご用意ください。(サーバー3-4台になるかもしれません)

## 参考資料
- [HPE Ezmeral Unified Analytics Software Documentation](https://support.hpe.com/hpesc/public/docDisplay?docId=a00eaf10hen_us&page=GetStarted/get-started.html)


## 事前準備
### インストーラーとOVAテンプレート入手
作業PCに以下のテンプレートとスクリプトをダウンロードしてください。もしくはHPE Software Centerから最新を入手してください。

- [ezua_k8s_q2_airgapped_testing.ova](https://hpe-ezua-pre-release-042023.s3.us-east-2.amazonaws.com/ezua_k8s_q2_airgapped_testing.ova)
- [start_ezua_installer_ui.sh](https://hpe-ezua-pre-release-042023.s3.us-east-2.amazonaws.com/start_ezua_installer_ui.sh)

### VMware設定
次にVMware側の設定をします。vCenterにログインしてください。

#### クラスタ作成
２つのクラスタを作成します。

- EZAF-MGMT-CLUSTER: Kuberentes Masterがデプロイされる
- EZAF-WORKER-CLUSTER: Kubernetes Workerがデプロイされる

各vSphereをそれぞれのクラスタにアサインしてください。

![](pics/vCenter_cluster.png)

#### VMフォルダ作成
お好きなUnified Analyticsで使用するVMフォルダを作ってください。本環境では*ua*としています。

![](pics/vCenter_folder01.png)

![](pics/vCenter_folder02.png)

![](pics/vCenter_folder03.png)

#### OVAのアップロード
最初に作業PCにダウンロードしたOVAテンプレートをVMフォルダにアップロードします。

![](pics/vCenter_ova_upload01.png)



![](pics/vCenter_ova_upload02.png)



![](pics/vCenter_ova_upload03.png)
作成したVMフォルダを選択します。VM名はそのままの名前にしときます。


![](pics/vCenter_ova_upload04.png)
適当なノードを選択します。


![](pics/vCenter_ova_upload05.png)


![](pics/vCenter_ova_upload06.png)
使用するデータストアとディスクフォーマットを選択します。


![](pics/vCenter_ova_upload07.png)
使用するネットワークを選択します。


![](pics/vCenter_ova_upload08.png)


#### VMのテンプレートの作成
OVAアップロードで作成されたVMをテンプレート化します。

![](pics/vCenter_ova_template01.png)
![](pics/vCenter_ova_template02.png)
![](pics/vCenter_ova_template03.png)


## インストール
### インストーラーの立ち上げ
Docker環境にログインして、事前にダウンロードしておいたスクリプトを実行します。

```
root@dns01:~/work/ua# chmod +x start_ezua_installer_ui.sh 

root@dns01:~/work/ua# ./start_ezua_installer_ui.sh 
Starting install of HPE Ezmeral Unified Analytics
Note: If lauching behind a corporate proxy, ensure HTTP_PROXY, HTTPS_PROXY, and NO_PROXY are set properly before continuing
Do you want to continue? [y/n]: y
Starting installer
Checking Docker Install
Checking for UI image. This may take a while to download.
Checking for installation image. This may take a few minutes to download.
ezaf_webapi
c32e04298d1c90cd3fec284a6eb3cd0a310f2764565aae354afeaf7e1fd8f7bd
HPE Ezmeral Software Unified Analytics is ready to install at: http://localhost:8080

```

Docker環境のport8080にwebブラウザでhttp接続します。以下の画面が表示されるので、*On-Prem(VMware)*を選択します。あとは画面に従って値を入力します。**vCenterから色々な値をコピペすると思いますが、くれぐれも意図しないスペースが入ってないことを注意してください。**

![](pics/installer01.png)
![](pics/installer02.png)
![](pics/installer03-1.png)
![](pics/installer03-2.png)
![](pics/installer04.png)
![](pics/installer05.png)
![](pics/installer06.png)

数十分待ちます。インストールログの見方は以下です。

```
$ docker ps
CONTAINER ID   IMAGE                                                COMMAND                  CREATED             STATUS             PORTS                                                 NAMES
ac38029dd57e   us.gcr.io/mapr-252711/hpe-ezua-orchestrator:v1.0.0   "./start.sh"             16 minutes ago      Up 16 minutes      3001/tcp, 0.0.0.0:3001->8000/tcp, :::3001->8000/tcp   ezaf-3001
c32e04298d1c   us.gcr.io/mapr-252711/hpe-ezua-installer-ui:v1.0.0   "node /usr/src/ezua-…"   About an hour ago   Up About an hour   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp             lucid_clarke


$ docker exec -it ac ls logs/
af_install-23-06-16_07:29:04.log

$ docker exec -it ac cat logs/af_install-23-06-16_07:29:04.log
```


![](pics/installer07.png)
![](pics/installer08.png)

しばらく待つとインストールが完了します。

インストーラー画面の左上に書いてあるIPをDNS登録します。Unbound DNSの例は以下の通りです。

```
server:
  local-zone: "hybrid-lab.local" redirect
  local-data: "hybrid-lab.local IN A 10.7.20.221"
  local-data: "hybrid-lab.local IN A 10.7.20.222"
```

### ライセンス適用
インストール終了画面下部のボタンを押すとライセンス適用の画面移動しますので、クラスターIDをコピーしてライセンスを取得してください。

![](pics/license01.png)

### ホーム画面
ライセンスを適用後、ログインしてホーム画面に移行します。

![](pics/home01.png)

