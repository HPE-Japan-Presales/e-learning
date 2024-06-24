# Azure Stack HCI 23H2


## 目次
### Azure Stack HCI (Azure Stack HCI OS 23H2 - )
- [Azure Stack HCIデプロイメント③：Azure Stack HCI OS を設定する](../installation03)  


#### Azure Stack HCI OSへの設定
※以下の作業はAzure Stack HCI全てのノード(今回ですと2つのサーバの両方)で実施してください。設定内容は適宜各ノードに合わせたパラメータを入力してください。

OSインストール直後の画面は以下になります。引き続きiLOへツールを使用してアクセスします。
![](pics/00.PNG)

画面上部「Keyboard」を選択後、「CTRL-ALT-DEL」を押します。
![](pics/01.PNG)

パスワード変更を求めるメッセージが表示されますのでOKを選択してEnterキーを押します。
![](pics/02.PNG)

パスワードを設定します。
`注意：パスワードには以下要件があります。しかしながらこの時点では要件を満たさないパスワードをAzure Stack HCI OSへ設定できてしまいますのでご注意ください。もし要件を満たさないパスワードを設定した場合は後に実行される事前チェックでエラーとなります。`

![](pics/password.PNG)
https://learn.microsoft.com/ja-jp/azure-stack/hci/deploy/deployment-install-os

![](pics/03.PNG)
![](pics/04.PNG)

Azure Stack HCI OSではおなじみのメニューが表示されました。
まずはOSにIPを設定します。PowerShellで設定するので 15 を入力してEnterキーを押します。
![](pics/05.PNG)

Get-NetAdapterコマンドでIPを設定するポートとifIndexを確認。
その情報を使用してNew-NetIPAddressコマンドでIPを設定します。
![](pics/06.PNG)

設定後、exitコマンドを実行してメインメニューに戻ります。DNSのIPはこちらのメニュー画面から設定を実施したいと思います。8 を入力してEnterキーを押します。
![](pics/08.PNG)

先ほどIPを設定したポートを選択するため、6 を入力してEnterキーを押します。
![](pics/07.PNG)

DNSを設定するため、2 を入力してEnterキーを押します。
![](pics/09.PNG)

DNSのIPを入力してEnterキーを数度押し設定を反映させます。
![](pics/11.PNG)

リモートデスクトップを有効化するため、7 を入力してEnterキーを押します。
![](pics/12.PNG)

2 を入力後、数回Enterキーを押します。
![](pics/14.PNG)

再度PowerShellを使って設定をするため、15 を入力してEnterキーを押します。
![](pics/16.PNG)

NTPを設定します。
```
w32tm /config /manualpeerlist:"<NTP Server>" /syncfromflags:manual /update
```
ホスト名を変更します。
```
Rename-Computer -NewName "<hostname>" -Force
```

ファイヤーウォールをOFFにします。
理由はこの後SPP(Service Pack for ProLiant)というドライバやファームウェア群を適用していくのですが、その際Smart Update Manager (SUM) というツールを使って適用します。そのSUMに対してhttpsでアクセスするのですが、このhttpsアクセスをファイヤーウォールでブロックされないようにするためです。本番環境ではない点と一時的にOFFにするだけですのでファイヤーウォール全てをOFFにしています。
```
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```
![](pics/17.PNG)

SPPを適用していきます。まずはSPPのメディアをiLOに仮想メディアとしてマウントします。
![](pics/18.PNG)
![](pics/19.PNG)

SPPがマウントされたディレクトリへ移動して以下スクリプトを実行します。
```
.\smartupdate
```
![](pics/20.PNG)
しばらくするとSUMが立ち上がります。
![](pics/21.PNG)

起動後、踏み台などのAzure Stack HCIに設定したIPへ到達できる端末上でブラウザを立ち上げます。ブラウザには以下のように63002ポートを使用して対象のIPへアクセスするようURLを入力します。SUMへのログインの際はAzure Stack HCI OSへ設定したAdministerユーザのログイン情報を使用します。
```
https://<Azure Stack HCI OSのIP>:63002
```
![](pics/22.PNG)

SUMへログイン後以下画面が表示されるので、「ローカルホストアップデート」を選択します。
![](pics/23.PNG)

デフォルト設定のまま画面下へ移動して「OK」を選択します。
![](pics/24.PNG)

ベースラインやインベントリの設定が実行されるので待ちます。
![](pics/25.PNG)
![](pics/26.PNG)

しばらくすると「次へ」ボタンを選択できるようになるので押します。
![](pics/27.PNG)

適用されるドライバやファームウェアが表示されます。
![](pics/28.PNG)

画面下へ移動してTPMに関する警告が出た場合はチェックを入れます。その後「展開」ボタンを押すとドライバやファームウェアのインストールが開始されます。
![](pics/29.PNG)
![](pics/30.PNG)

適用が完了すると以下画面のような画面が表示されるので画面下へ移動します。
![](pics/31.PNG)

「再起動」ボタンを選択してサーバを再起動させます。
![](pics/32.PNG)
![](pics/33.PNG)

サーバの再起動を以下の画面が表示されたら、マウントしていたSPPのメディアをアンマウントします。チェックを外せばアンマウントされます。
<span style="color:red;">注意：必ずアンマウントをしてください。忘れるとデプロイ時に仮想メディアを不明なディスクとして認識して後々エラーになる可能性があります。</span>
![](pics/34.PNG)

OFFにしていたファイヤーウォールをONに戻します。
```
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
```
![](pics/35.PNG)

Hyper-Vロールをインストールします。
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
![](pics/36.PNG)

インストール後、OS再起動させる旨の確認が入るのでYを選択してサーバを再起動させます。
![](pics/37.PNG)
