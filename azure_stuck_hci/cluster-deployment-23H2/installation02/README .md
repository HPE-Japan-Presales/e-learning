# Azure Stack HCI 23H2


## 目次
### Azure Stack HCI (Azure Stack HCI OS 23H2 - )
- [Azure Stack HCIデプロイメント②：Azure Stack HCI OS をインストールする](installation02)  

#### インストール前の事前準備
インストール前にAzure Stack HCI バージョン 23H2 のデプロイの前提条件を以下URLで確認します。
https://learn.microsoft.com/ja-jp/azure-stack/hci/deploy/deployment-prerequisites

今回の環境ではシステム要件の「セキュアブート」を有効化する必要がありました。
https://learn.microsoft.com/ja-jp/azure-stack/hci/concepts/system-requirements-23h2

![](pics/youken.PNG)

##### セキュアブートの有効化
作業は踏み台のWindows OSから実施します。この踏み台に「HPE iLO スタンドアロンリモートコンソール」をインストールして、このツールを利用してサーバの操作をしていきます。

※以下の作業はAzure Stack HCI全てのノード(今回ですと2つのサーバの両方)で実施してください。

ツールで対象のiLOへ接続後、サーバの電源をONにします。しばらくすると以下画面が表示されるのでF9キーを押します。
![](pics/02.PNG)

しばらくすると以下画面が表示されるので「System Configuration」を選択します。
![](pics/03.PNG)

続いて「BIOS/Platform Configuration (RBSU)」を選択します。
![](pics/04.PNG)

「Server Security」を選択します。
![](pics/05.PNG)

「Server Boot Settings」を選択します。
![](pics/06.PNG)

「Attempt Secure Boot」のプルダウンをDisabled→Enabledへ変更します。
その後画面右下「F12: Save and Exit」を選択すると設定を有効化するためサーバが再起動します。
![](pics/07.PNG)
Azure Stack HCIのノード全てでこの作業を実施します。

#### Azure Stack HCI OSのインストール
※以下の作業はAzure Stack HCI全てのノード(今回ですと2つのサーバの両方)で実施してください。
※今回の手順には記載していませんが必要に応じてAzure Stack HCI OSのインストール先となるボリュームのRAID設定を実施します。今回の環境ですとHDD x3本を使用してRAID5を構成してHDD x1本をスペアディスクとして設定しています。

ツールを使用してiLOへ接続します。
「Virtual Drives」を選択後、「Image File CD-ROM/DVD」にチェックを入れます。
![](pics/00.PNG)

Azure Stack HCI OSのISOイメージを選択して「開く」を押します。
※Azure Stack HCI OSはAzure PortalのAzure Stack HCIのメニューからダウンロードしています。詳しくはこちらをご参照ください。
https://learn.microsoft.com/ja-jp/azure-stack/hci/deploy/download-azure-stack-hci-23h2-software
![](pics/01.PNG)

サーバを起動させて「F11」を選択します。
![](pics/08.PNG)

ブートメニューの選択画面になるので、「iLO Virtual USB 3: iLO Virtual CD_ROM」を押します。
![](pics/09.PNG)

ISOメディアからの起動が開始されるので何かしらのキーボードのキーを押します。
![](pics/10.PNG)

ISOメディアのデータの読み込みが開始されます。
![](pics/11.PNG)

Azure Stack HCI OSのインストール画面が表示されるので適宜メニューを選択して「Next」を押します。(今回は英語を言語として選択しています。)
![](pics/12.PNG)

画面真ん中「Install now」を選択します。
![](pics/13.PNG)

以下画面が表示されるので少し待ちます。
![](pics/14.PNG)

チェックを入れて「Next」を選択します。
![](pics/15.PNG)

「Custom: Install the newer version of Azure Stack HCI only (advanced)」を選択します。
![](pics/16.PNG)

OSをインストールする対象のDiskを選択して「Next」を押します。今回はHDD を使用したRAIDボリュームを選択します。他に見えているディスクはSSD x4本になります。
![](pics/17.PNG)

インストールが開始されてます。インストール完了後に自動でサーバが再起動されます。
![](pics/18.PNG)

Azure Stack HCIのノード全てでこの作業を実施します。
