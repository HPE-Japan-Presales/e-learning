# vSphere with Tanzu / Tanzu K8s Grid Service Installation
NVAIEの基盤の下地を作るため、
vSphere with Tanzuの環境を作っていきます。

## TKGs構築④ - コンテンツライブラリの用意&ストレージポリシーの有効化
vSphere 7.0U1以降でスーパーバイザークラスタ（ワークロード管理）を有効化するには、<br>
Tanzu K8s Grid OVF用のコンテンツライブラリの事前作成が必要です。

TKGmの場合、一般的なVMテンプレートからノードをデプロイする必要があるみたいですが、TKC(TKGs内のTanzu K8s Cluster)ではコンテンツライブラリのOVFテンプレートを利用可能なため、こちらを実施します。<br>

**参考ドキュメント**<br>
ーーーーーー<br>
[Tanzu Kubernetes リリース のコンテンツ ライブラリの作成と管理](https://docs.vmware.com/jp/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-209AAB32-B2ED-4CDF-AE62-B0FAD9D34C2F.html)<br>
ーーーーーー<br>

#### コンテンツライブラリ作成

![](pics/pic01.png)<br>
vCenterメニューより、

![](pics/pic02.png)
コンテンツライブラリの名称と、vCenterサーバーを選択

![](pics/pic03.png)
”サブスクライブ済みコンテンツライブラリ”を選択
サブスクリプションURLはドキュメント指定の、<br>
”https://wp-content.vmware.com/v2/latest/lib.json”
を入力。<br>
コンテンツダウンロードオプションはデフォルトの”今すぐ”を選択

![](pics/pic04.png)

![](pics/pic05.png)
今回は特にセキュリティポリシーの適用はなしで実施

![](pics/pic06.png)
コンテンツライブラリ用のデータストアを選択

![](pics/pic08.png)
コンテンツライブラリ作成完了

![](pics/pic09.png)
コンテンツのダウンロードを”今すぐ”で実行すると、
K8sバージョンごとに自動的にすべてダウンロードされる。
必要バージョンが明確な場合は、選択してダウンロードしたほうがデータストアも逼迫させずに実行できるので有用かと。

#### ストレージポリシーの有効化

Tanzu環境では使用するデータストアを直接指定せず、ストレージポリシーを利用するため、あらかじめ設定しておく必要あります。<br>
はじめにvSphereタグを作成します。<br>
![](pics/pic10.png)
<br>
vCenterメニューより、"メニュー" > "タグとカスタム属性"

![](pics/pic11.png)
”新規”を選択、”タグの作成”　＞　”新しいカテゴリの作成”

![](pics/pic12.png)

![](pics/pic13.png)
タグの名前：”tkgs-demo”<br>
カテゴリ："datastore-category"

![](pics/pic14.png)

次にデータストアに作成したタグを割り当てます。

![](pics/pic15.png)
タグ付けするデータストアのサマリー画面より、”タグの割り当て”を選択

![](pics/pic16.png)

![](pics/pic17.png)
この例では、"windows-nfs-2"というデータストアに作成したタグを割り当ててます。

次に仮想マシンポリシーを作成し、TKGsで作成される仮想マシンに対しても同様のストレージポリシーが適用されるように設定します。
![](pics/pic18.png)
メニューより、”ポリシーおよびプロファイル”を選択

![](pics/pic19.png)
"作成"をクリック後、”vm-storage-policy-tkgs”を作成

![](pics/pic20.png)
”ポリシー構造”　＞　”データストア固有ルール”　＞　”タグベースの配置ルールを有効化”にチェック

![](pics/pic21.png)
タグカテゴリ：”datastore-category”<br>
使用量オプション：”以下のタグ付けをされたストレージを使用”<br>
タグ：”tkgs-demo”

![](pics/pic22.png)
タグを付与したデータストアが表示されることを確認

![](pics/pic23.png)
![](pics/pic24.png)
仮想マシンストレージポリシーが作成されたことを確認。<br>
これでコンテンツライブラリの準備が整いました。
