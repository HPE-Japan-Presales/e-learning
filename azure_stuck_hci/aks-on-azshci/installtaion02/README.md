# Azure Kubernetes Service on Azure Stack HCIを使ってみる
マネージドK8sのAzure K8s ServiceがオンプレHCIの上でも対応しましたので、下記にその利用方法をまとめます。


-----
**※** 参照ドキュメントはこちら：<br>
- [クイック スタート: Windows Admin Centerを使用して AKS ハイブリッドでローカル Kubernetes クラスターを作成する](https://learn.microsoft.com/ja-jp/azure/aks/hybrid/create-kubernetes-cluster)
-----

## ② K8sクラスターの作成
Windows Admin Center管理画面より、> クラスターマネージャー > Azure Kubernetes サービス を選択し、次のステージに進みます。
![](pics/pic01.png)

バックエンドでAzure Arcの機能を利用したほうが、Azure Portalから一元的に管理・運用が可能となるため、必要な情報を入力していきます。
![](pics/pic02.png)

ワークロード用のK8sクラスターの名前、バージョンを選択し、
![](pics/pic03.png)

ワークロード用K8sクラスターを構成するための、ロードバランサーノードと、コントロールプレーン用のマスターノードのスペックを設定しておきます。（Azure IaaSと同様のサイズ感です）
![](pics/pic04.png)

続いて、ワーカーノードのノードプールの設定を実施します。<br>
Linuxワーカーなのか、Windowsワーカーなのか、またノード数とノード当たりのスペックを設定します。<br>
Podのスケジューリングを支援するテイント機能もGUIで設定することが可能です。
![](pics/pic05.png)

今回はLinuxプールを作成してみます。
![](pics/pic06.png)

次にActive Directoryを利用したユーザー認証をするか否かの設定をします。
![](pics/pic07.png)

今回はAD連携はせずに進めます。
![](pics/pic08.png)

次にネットワークの設定をします。事前に用意した仮想ネットワークを追加し、K8sのCNIプラグインや、ロードバランサー手法などを選択します。
![](pics/pic09.png)

設定した情報を確認します。
![](pics/pic10.png)

"作成"をクリックしてワークロード用K8sクラスターを作成します。
![](pics/pic11.png)

エラーが出てますが作成できてそう、、、。
![](pics/pic12.png)

"Azure Kubernetes サービス" > "Kubernetesクラスター"を見てみると、「my-workload-cluster」が作成できてますね。
![](pics/pic13.png)

「my-workload-cluster」を覗いてみると、きちんとAzure Arc登録されていて、K8sのバージョン情報等も確認できます。
![](pics/pic14.png)

主要なメトリクスもこちらの画面から確認可能です。
![](pics/pic15.png)

拡張機能や、チュートリアルなどはこのように見えます。
![](pics/pic16.png)

![](pics/pic17.png)

クラスター内に作成された仮想マシンを確認してみると、<br>
管理クラスターノード、ワークロード用K8sのロードバランサーノード、マスターノード、ワーカーノードと、各役割を担う仮想マシンが作成されていることがわかります。
![](pics/pic18.png)

Azure Portal上からみると、management-cluster、workload-clusterともに作成されていることがわかります。
![](pics/pic19.png)

![](pics/pic20.png)

![](pics/pic21.png)
Management-Cluster、Workload-CLusterともにWindows Admin Centerでの表示内容と類似した管理画面となってます。
