# NVIDIA A100 をVMware Tanzu Kubernetes Gridにパススルーして使ってみる
TKG上でA100をパススルーして、Nvidia GPU Operatorを使ってみました。なお、TKGノードのOSはUbuntu20.04を選択しています。

## 改訂履歴

| バージョン | 日付 | 改訂者 | 更新内容 |
| :---: | :---: | :---: | :---: |
| 0.1 | 2021.11.19 | [Taku Kimura @HPE Japan Presales](taku.kimura@hpe.com) | 初版発行 |
|  |  |  |  |

## 目標
- Nvidia GPU Operatorを使ってTKG上でA100を使えるか確かめる
- Multi Instance GPU (MIG)が使えるかも確かめる


## 手順
### GPUをk8s workerにパススルーする
既にデプロイされたk8s workerに対してGPUをパススルーで提供しようと思います。

今回GPUを与えるk8sクラスタの構成は以下です。

```
$ kubectl get node
NAME                                              STATUS   ROLES                  AGE   VERSION
takluster-gpu-passthrough-control-plane-g9wkj     Ready    control-plane,master   17m   v1.20.5+vmware.2
takluster-gpu-passthrough-control-plane-hwmsk     Ready    control-plane,master   13m   v1.20.5+vmware.2
takluster-gpu-passthrough-control-plane-j5lvk     Ready    control-plane,master   21m   v1.20.5+vmware.2
takluster-gpu-passthrough-md-0-65c89dd9db-8fqdb   Ready    <none>                 17m   v1.20.5+vmware.2
takluster-gpu-passthrough-md-0-65c89dd9db-9r4f5   Ready    <none>                 17m   v1.20.5+vmware.2
takluster-gpu-passthrough-md-0-65c89dd9db-xhbc7   Ready    <none>                 17m   v1.20.5+vmware.2
```

*takluster-gpu-passthrough-md-0-65c89dd9db-xhbc7*にGPUを渡しています。