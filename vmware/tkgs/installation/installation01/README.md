# vSphere with Tanzu / Tanzu K8s Grid Service Installation
NVAIEの基盤の下地を作るため、
vSphere with Tanzuの環境を作っていきます。

## vSphere with Tanzu システム要件
NVAIE環境を作るうえで、まずはvSphere with Tanzuの要件を整理したいと思います。

**参考**<br>
ーーーーーー<br>
[vSphere with Tanzuの設定と管理](https://docs.vmware.com/jp/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-152BE7D2-E227-4DAA-B527-557B564D9718.html)<br>
[vSphere with Tanzu Quick Start Guide V1a](https://core.vmware.com/resource/vsphere-tanzu-quick-start-guide-v1a#_Toc53677530)<br>
[vSphere クラスタで vSphere with Tanzu を構成するための前提条件](https://docs.vmware.com/jp/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-EE236215-DA4D-4579-8BEB-A693D1882C77.html)<br>
[Tanzu Kubernetes Gird サービス v1alpha2 APIを使用するための要件](https://docs.vmware.com/jp/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-0CA8BF39-0D7E-4335-9D5B-7C80ED90D4D8.html)<br>
[HAProxyロードバランサで使用するスーパーバイザークラスタのvSphere Distributed Switchの作成](https://docs.vmware.com/jp/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-A3EEF5E4-3FAB-4193-B924-5579716D112A.html#GUID-A3EEF5E4-3FAB-4193-B924-5579716D112A)<br>
ーーーーーー<br>

![](pics/pic01.png)
システム要件：
- 3台以上のESXiホスト
- vSANはMustではない
- ネットワークはNSX-T構成と、vDS+HAProxyの2パターン

![](pics/pic02.png)
一番手っ取り早いのはNSX-Tを使わず、vDS + HAProxy環境下でのTKGsのため、今回はこちらを採用します。

ざっくりの手順としては、<br>
①vDS作成<br>
②共有データストアの作成<br>
③ロードバランサ用HA Proxyの用意<br>
④コンテンツライブラリの用意&ストレージポリシーの有効化<br>
⑤ワークロードの管理の有効化（Supervisor Clusterの作成）<br>
⑥NameSpaceや子Clusterを作成してK8sを使えるようにしていく<br>
に沿って実施していこうと思います。

## TKGs構築① - vDS作成　
vDSを作成していきます。

![](pics/pic03.png)
”DataCenter" > "Distributed Switch"

![](pics/pic04.png)
vDSの名前、配置場所を入力、設定

![](pics/pic05.png)
バージョンの選択

![](pics/pic06.png)
ポートグループ情報等記載。
このPrimaryポートグループはスーパーバイザークラスタのプライマリワークロードネットワーク（K8s制御プレーン仮想マシントラフィックの処理）として利用します。

![](pics/pic07.png)<br>
設定の確認

![](pics/pic08.png)
作成した”分散仮想スイッチ”に移動し、右クリック。
"分散ポートグループ" > "新規分散ポートグループ"を選択します。

![](pics/pic09.png)
今回はデフォルトの設定で作成します。

![](pics/pic10.png)
今回はすべてのK8s名前空間用のネットワークを一つの分散ポートグループで実施する想定で1つのみ作成しておりますが、名前空間毎にネットワークを隔離したい場合は、ポートグループを複数作成して実施することも可能です。

![](pics/pic11.png)
スーパーバイザークラスターとして構成するvSphereクラスタのホストをvDSに追加します。
vDSを右クリック、"ホストの追加と管理"を選択します。

![](pics/pic12.png)
”ホストの追加”を選択

![](pics/pic13.png)
追加するホストを選択

![](pics/pic14.png)
各ホストから物理NICを選択し、vDS上のアップリンクに割り当てる

![](pics/pic15.png)

![](pics/pic16.png)

![](pics/pic17.png)
特に移行するものがなければ次に進める

![](pics/pic18.png)

![](pics/pic19.png)
vDSの準備ができました。

![](pics/pic20.png)
ただし、後述する["TKGs構築① - ワークロード管理の有効化"において"](nvidia-ai-enterprise/installation05)、<br>
vDSのポートグループ名で "**-**" が使用できないことが判明したため、最終構成は下記に変更します。


![](pics/pic21.png)
DPortGroup_mgmt → 管理ネットワーク <br>
DPortGroup_primary → ワークロードネットワーク①<br>
DPortGroup_secondary → ワークロードネットワーク②<br>
の構成でこの先を進めます。
