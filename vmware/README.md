# VMware
このレポジトリではVMware関連のナレッジをまとめています。

## 目次
### Tanzu Kubernetes Grid Multicloud(TKGm)
- [HPE ProLiantにインストール](tkgm/installation)  
Tanzu Kubernetes Grid(TKG)のインストール方法

- [使ってみる01](tkgm/instruction01)  
NginxのデプロイやPVの宣言、NSXがない環境でのMetal LBやIngress Controller導入などについて

- [HPE Nimble Storageと連携してみる](tkgm/nimble)  
HPE Storageと連携方法について

- [トラブルシュート系 ](tkgm/trouble_shoot)  
検証の際に出会った問題等

### vSphere with Tanzu / Tanzu K8s Grid Service(TKGS)
- [TKGs基本セットアップ](tkgs/installation)  
  vSphere with K8s / Tanzu K8s Grid Service 初期セットアップ手順
- [TKGs - Supervisor Cluster作成してみる](tkgs/s_cluster)  
  K8sワークロードの有効化、Supervisor Cluster(管理用Kubernetes Cluster)の作成手順
- [TKGs - k8s Cluster作成してみる](tkgs/k8s_cluster)  
  ユーザー用Kubernetes Clusterの作成手順
- [Nvidia AI Enterprise環境をセットアップしてみる](tkgs/nvidia-ai-enterprise)  
  Nvidia AI Enterprise環境にするための初期セットアップ手順
- [GPUをアサインしてみる](tkgs/nvidia-ai-enterprise/k8s)  
  Nvidia AI Enterprise環境でk8sクラスタにGPUのアサイン手順

### vGPU Software(GRID)
- [AIワークロード用の仮想GPU利用について](vgpu/instruction01)  
vGPU Software / NVIDIA AI Enterpriseの整理について

- [仮想マシンでvGPUが使えるようになるまで](vgpu/installation01)  
vGPU Managerのインストール、インストールイメージ入手、vCenter側での設定について

- [vGPU用ライセンスサーバーを構築する](vgpu/installation02)  
ライセンスサーバー構築と、vGPUライセンス管理について