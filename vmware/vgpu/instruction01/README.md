# AIワークロード用の仮想GPU利用について
vGPU Software v13.0より仮想GPUの利用体系が変更になり、<br>
ハイパーバイザーにより使用できるソフトウェアとエディションが異なるので注意が必要です。<br>
従来のvGPU Software（vWS/vPC/vApps/vCS）とNVIDIA AI Enterprise(vCS)用途で異なるvGPU Managerとライセンス体系を取るため、下記にまとめてみます。


## vGPU Software / NVIDIA AI Enterprise

主要ハイパーバイザー毎のvGPU利用の整理をしてみます。

| ハイパーバイザー | エディション | 必要なソフトウェア |
| :---: | :---: | :---: |
| RHEL with KVM / RHEV | All | vGPU Software |
| Ubuntu with KVM | All | vGPU Software |
| Nutanix AHV | All | vGPU Software |
| VMware | vWS/vPC/vApps | vGPU Software |
| VMware | vCS | NVIDIA AI Enterprise |
| Azure Stack HCI | vWS/vPC/vApps  | vGPU Software |
|  |  |  |

- ちなみにWindowsはDDAのみ、いわゆるvGPUとしての使い方だったRemoteFXは現行OSバージョンでは廃止、今後のロードマップに同等機能追加の話はあるみたいですが時期は未定。
- VMware利用時は少々複雑で、下記の通りvCS用途利用時にはNVIDIA AI Enterpriseに統合されています。[by vGPU VMware support docs](https://docs.nvidia.com/grid/latest/product-support-matrix/index.html#abstract__vcompute-gpu-support-limitations)<br>

  - *NVIDIA Virtual Compute Server (vCS) is not supported on VMware vSphere ESXi. C-series vGPU types are not available. Instead, vCS is supported with NVIDIA AI Enterprise.*


NVIDIA AI Enterpriseでの実装方法をまとめてみると

- **with VMware**
  - 仮想ベース：VMware + Container Runtime(Docker etc) + Container(NGC)
  - コンテナベース：vSphere with Tanzu / Tanzu K8s Grid Service + Container(NGC)

- **with Red Hat**
  - 仮想K8s ベース：VMware + RHEL/RHCOS + OpenShift + Container(NGC)
  - ベアメタルK8sベース：RHEL/RHCOS + OpenShift + Container(NGC)
  - ベアメタルベース：RHEL/RHCOS + Container Runtime(Podman) + Container(NGC)

と結構選択肢が多いです。

#### 参考ドキュメント
ーーーーーー<br>
**各エディション（vWS/vPC/vApps/vCS）の違い等はこちら**<br>
[NVIDIA Virtual GPU Software Documentation - Display Resolution](
https://docs.nvidia.com/grid/latest/grid-vgpu-user-guide/index.html#physical-gpu-display-resolution-handling)

**vGPU Softwareサポートマトリックス**<br>
[vGPU Software product support matrix:](https://docs.nvidia.com/grid/latest/product-support-matrix/index.html)

**NVIDIA AI Enterpriseサポートマトリックス**<br>
[NVIDIA AI Enterprise product support matrix:](https://docs.nvidia.com/ai-enterprise/latest/product-support-matrix/index.html)

**NutanixプラットフォームでのvGPU対応**<br>
[NVIDIA vGPU on Nutanix:](https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2046-vGPU-on-Nutanix:TN-2046-vGPU-on-Nutanix)  
ーーーーーー<br>
