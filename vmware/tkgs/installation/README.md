# VMware
このレポジトリではvSphere with Tanzu / Tanzu K8s Grid Service(TKGs)関連のナレッジをまとめています。

## 目次
### vSphere with Tanzu / Tanzu K8s Grid Service(TKGs)
- [TKGs構築① - vDS作成](installation01)  
NginxのデプロイやPVの宣言、NSXがない環境でのMetal LBやIngress Controller導入などについて

- [TKGs構築② - 共有データストアの作成](installation02)<br>
クラスターを構成する各ホストがアクセス可能な共有データストアを用意

- [TKGs構築③ - ロードバランサ用のHA Proxyの用意](installation03)<br>
NSX-Tによるロードバランサーの代わりとなるHA Proxy仮想アプライアンスをデプロイ

- [TKGs構築④ - コンテンツライブラリの用意＆ストレージポリシーの有効化](installation04)<br>
Tanzu K8s Grid OVF用のコンテンツライブラリの事前作成
