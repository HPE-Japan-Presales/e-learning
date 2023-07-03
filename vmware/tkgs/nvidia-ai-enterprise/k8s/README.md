# Nvidia AI Enterprise環境でk8sクラスタにGPUをアサインしてみる


## 検証バージョン
- vSphere 7.0U3
- vCenter 7.0.3
- Nvidia A100 40G

## 前提
- TKGSの初期セットアップが完了していること
- Nvidia AI Enterprise環境の初期セットアップが完了していること
- TKGS Supervisor Clusterの作成が完了していること
- Namespaceのセットアップが完了していること
- クライアントPCにKubectlとKubectl Vsphere Pluginがインストール済みであること
- クライアントPCにHELMがインストール済みであること
- Contents Libraryの設定が完了していること

## 手順
### GPUが付与されたVM Class作成
本検証環境で事前に用意されている*tak*のNamespace内にGPUが付与されているVM Classを作成します。

*Manage VM Classes*を選択します。
![](pics/vmclass01.png)

さらに、*Manage VM Classes*を選択します。
![](pics/vmclass02.png)


*CREATE VM CLASS*を選択します。
![](pics/vmclass03.png)

ナビゲーションに従い、必要な値を入力します。*PCI Devices*のチェックを有効にすることを忘れないでください。
![](pics/vmclass04.png)

PCI Deviceとして*Nvidia vGPU*を選択します。
![](pics/vmclass05.png)

A100なので*MIG*を選択してみます。
![](pics/vmclass06.png)

設定値が間違っていないことを確認して*Finish*ボタンを押します。
![](pics/vmclass07.png)

作成した*gpu01*というVM ClassをNamespaceにアサインします。
![](pics/vmclass08.png)

### GPU用のイメージ取得
[こちらのシステム要件](https://docs.vmware.com/jp/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-9A458012-B6A8-4A8B-9052-23D9FD164420.html#GUID-9A458012-B6A8-4A8B-9052-23D9FD164420__section_i1b_25g_crb)に書いてある通り、GPUを持たせるノードには特定のイメージを持たせる必要があります。事前にコンテンツライブラリに配置しておく必要があります。今回は*ob-18691651-tkgs-ova-ubuntu-2004-v1.20.8---vmware.1-tkg.2*のイメージ使用しています。

### k8sクラスタ作成用マニュフェスト
GPUリソースを持ったk8sクラスタを作成してみます。

先ほど作成した、GPUを持ったVM ClassがSupervisor Clusterから正常に見えるか確認します。

```bash
$ k get vmclass
NAME                 CPU   MEMORY   AGE
best-effort-large    4     16Gi     10d
best-effort-medium   2     8Gi      10d
best-effort-small    2     4Gi      10d
gpu01                4     16Gi     28s
```

次にk8sクラスタ作成用のマニフェストを用意します。ファイル名は**tkg-gpu-cluster01**とします。コンテナランタイムはcontainerdを使っている様で、イメージが置かれる*/var/lib/containerd*に少しボリュームを持たせています。Tanzuのイメージは[デフォルトで16GB](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-7351EEFF-4EF0-468F-A19B-6CEA40983D3D.html#:~:text=For%20Tanzu%20Kubernetes%20releases%2C%20the%20disk%20size%20is%2016GB.)しかない様です。

```yaml
apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
metadata:
  name: tkg-gpu-cluster01
  namespace: tak
spec:
  topology:
    controlPlane:
      replicas: 1
      vmClass: best-effort-small
      storageClass: tak-tkgs-storage
      tkr:  
        reference:
          name: v1.20.8---vmware.1-tkg.2
      volumes:
      - capacity:
          storage: 30Gi
        mountPath: /var/lib/containerd
        name: containerd
        storageClass: tak-tkgs-storage
    nodePools:
    - name: tkg-gpu-cluster01-nodepool-01
      replicas: 1
      vmClass: best-effort-small
      storageClass: tak-tkgs-storage
      tkr:  
        reference:
          name: v1.20.8---vmware.1-tkg.2 
      volumes:
      - capacity:
          storage: 30Gi
        mountPath: /var/lib/containerd
        name: containerd
        storageClass: tak-tkgs-storage
    - name: tkg-gpu-cluster01-nodepool-02
      replicas: 1
      vmClass: gpu01
      storageClass: tak-tkgs-storage
      tkr:  
        reference:
          name: v1.20.8---vmware.1-tkg.2
      volumes:
      - capacity:
          storage: 100Gi
        mountPath: /var/lib/containerd
        name: containerd
        storageClass: tak-tkgs-storage
```

マニュフェストを適用します。

```bash
$ k apply -f tkg-gpu-cluster01.yaml
tanzukubernetescluster.run.tanzu.vmware.com/tkg-gpu-cluster01 created

$ k get tkc
NAME                CONTROL PLANE   WORKER   TKR NAME                   AGE   READY   TKR COMPATIBLE   UPDATES AVAILABLE
tkg-gpu-cluster01   1               2        v1.20.8---vmware.1-tkg.2   10m   True    True

```

クラスタがReadyになったら、KubeConfigを取得します。

```bash
$ kubectl-vsphere login --server=192.168.3.43 --insecure-skip-tls-verify --tanzu-kubernetes-cluster-name tkg-gpu-cluster01 --tanzu-kubernetes-cluster-namespace tak

Username: administrator@vsphere.local
KUBECTL_VSPHERE_PASSWORD environment variable is not set. Please enter the password below
Password:
Logged in successfully.

You have access to the following contexts:
   192.168.3.43
   tak
   tak-192.168.4.129
   tkg-cluster01
   tkg-gpu-cluster01

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`

$ k config use-context tkg-gpu-cluster01
Switched to context "tkg-gpu-cluster01".
```

軽くクラスタの状況を確認します。

```bash
$ k get node --show-labels
NAME                                                              STATUS   ROLES                  AGE     VERSION            LABELS
tkg-gpu-cluster01-control-plane-965cz                             Ready    control-plane,master   10m     v1.20.8+vmware.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=tkg-gpu-cluster01-control-plane-965cz,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,run.tanzu.vmware.com/kubernetesDistributionVersion=v1.20.8_vmware.1-tkg.2
tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-01-6k7cs-6bbck47m8   Ready    <none>                 5m31s   v1.20.8+vmware.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-01-6k7cs-6bbck47m8,kubernetes.io/os=linux,run.tanzu.vmware.com/kubernetesDistributionVersion=v1.20.8_vmware.1-tkg.2
tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-02-czl9w-75fbdmppm   Ready    <none>                 5m32s   v1.20.8+vmware.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-02-czl9w-75fbdmppm,kubernetes.io/os=linux,run.tanzu.vmware.com/kubernetesDistributionVersion=v1.20.8_vmware.1-tkg.2

$ k get pod -A
NAMESPACE                      NAME                                                                              READY   STATUS    RESTARTS   AGE
kube-system                    antrea-agent-85trd                                                                2/2     Running   0          9m38s
kube-system                    antrea-agent-9mwdl                                                                2/2     Running   0          5m1s
kube-system                    antrea-agent-xs8xq                                                                2/2     Running   0          5m
kube-system                    antrea-controller-d67874fbd-zc6nf                                                 1/1     Running   0          9m39s
kube-system                    antrea-resource-init-84c5bc8bbb-ljf7b                                             1/1     Running   0          9m39s
kube-system                    coredns-b9c799994-4vzcw                                                           1/1     Running   0          8m32s
kube-system                    coredns-b9c799994-tktxv                                                           1/1     Running   0          9m36s
kube-system                    docker-registry-tkg-gpu-cluster01-control-plane-965cz                             1/1     Running   0          9m25s
kube-system                    docker-registry-tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-01-6k7cs-6bbck47m8   1/1     Running   0          4m59s
kube-system                    docker-registry-tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-02-czl9w-75fbdmppm   1/1     Running   0          5m
kube-system                    etcd-tkg-gpu-cluster01-control-plane-965cz                                        1/1     Running   0          9m25s
kube-system                    kube-apiserver-tkg-gpu-cluster01-control-plane-965cz                              1/1     Running   0          9m25s
kube-system                    kube-controller-manager-tkg-gpu-cluster01-control-plane-965cz                     1/1     Running   0          9m25s
kube-system                    kube-proxy-8tj4m                                                                  1/1     Running   0          5m1s
kube-system                    kube-proxy-b29hq                                                                  1/1     Running   0          9m36s
kube-system                    kube-proxy-bgnlr                                                                  1/1     Running   0          5m
kube-system                    kube-scheduler-tkg-gpu-cluster01-control-plane-965cz                              1/1     Running   0          9m25s
kube-system                    metrics-server-5fdfb857c7-wck7k                                                   1/1     Running   0          9m44s
vmware-system-auth             guest-cluster-auth-svc-dqphg                                                      1/1     Running   0          8m53s
vmware-system-cloud-provider   guest-cluster-cloud-provider-976b4bcdd-cwjmf                                      1/1     Running   0          9m45s
vmware-system-csi              vsphere-csi-controller-5ccc98fb8f-qw57w                                           6/6     Running   0          9m45s
vmware-system-csi              vsphere-csi-node-5kqt2                                                            3/3     Running   0          9m38s
vmware-system-csi              vsphere-csi-node-wd684                                                            3/3     Running   0          5m1s
vmware-system-csi              vsphere-csi-node-wj9q4                                                            3/3     Running   0          5m

$ k get ns
NAME                           STATUS   AGE
default                        Active   10m
kube-node-lease                Active   10m
kube-public                    Active   10m
kube-system                    Active   10m
vmware-system-auth             Active   9m52s
vmware-system-cloud-provider   Active   9m51s
vmware-system-csi              Active   9m50s

$ k get node -o wide
NAME                                                              STATUS   ROLES                  AGE     VERSION            INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
tkg-gpu-cluster01-control-plane-965cz                             Ready    control-plane,master   13m     v1.20.8+vmware.1   192.168.4.46   <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.4.6
tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-01-6k7cs-6bbck47m8   Ready    <none>                 8m28s   v1.20.8+vmware.1   192.168.4.44   <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.4.6
tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-02-czl9w-75fbdmppm   Ready    <none>                 8m29s   v1.20.8+vmware.1   192.168.4.47   <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.4.6
```

Nvidia用GPU Plugin的なものが入っていないようなので、手動で入れます。

### Nvidia GPU Operatorのインストール
Nvidia AI Enterprise用の[公式手順](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/install-gpu-operator-nvaie.html#installing-gpu-operator)に沿ってGPU Operatorをインストールしていきます。

まず、GPU入りk8sクラスタ内にNamespaceを作成します。

```bash
$ k create namespace gpu-operator
namespace/gpu-operator created

$ k get ns
NAME                           STATUS   AGE
default                        Active   17h
gpu-operator                   Active   4s
kube-node-lease                Active   17h
kube-public                    Active   17h
kube-system                    Active   17h
vmware-system-auth             Active   17h
vmware-system-cloud-provider   Active   17h
vmware-system-csi              Active   17h
```

**gridd.conf**というファイルを作成します。今回はオンプレのNvidiaライセンスサーバーを作成しているので、その情報を記載します。

```bash
$ touch gridd.conf
$ vi gridd.conf
ServerAddress=192.168.3.32
ServerPort=7070
FeatureType=1
```

NLS client license tokenなるものを[NGC](https://ngc.nvidia.com/signin)、もしくはオンプレのライセンスサーバーから取得します。Nvidia AI Enterpriseのライセンスがないと取得できないと思いますので注意してください。Tokenは**client_configuration_token.tok**というファイル名で保存します。

k8s内の*gpu-operator*ネームスペースにconfigmapを作成します。

```bash
$ k create configmap licensing-config -n gpu-operator --from-file=gridd.conf --from-file=client_configuration_token.tok
configmap/licensing-config created

$ k get cm -n gpu-operator
NAME               DATA   AGE
kube-root-ca.crt   1      6m35s
licensing-config   2      30s

```

NGCのコンテナレジストリにアクセスできるようにするため、k8s内の*gpu-operator*ネームスペースにsecretを作成します。

```
$ k create secret docker-registry ngc-secret \
    --docker-server=nvcr.io/nvaie \
    --docker-username='$oauthtoken' \
    --docker-password='<YOUR-NGC-API-key>' \
    --docker-email='<YOUR-NGC-email-address>' \
    -n gpu-operator

$ k get secret -n gpu-operator
NAME                  TYPE                                  DATA   AGE
default-token-zwx9k   kubernetes.io/service-account-token   3      9m18s
ngc-secret            kubernetes.io/dockerconfigjson        1      14s

```

helmにrepoを登録します。

```bash
$ helm repo add nvaie --username='$oauthtoken' --password='<YOUR-NGC-API-key>' https://helm.ngc.nvidia.com/nvaie
"nvaie" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "nvidia" chart repository
...Successfully got an update from the "nvaie" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm search repo nvaie
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
nvaie/gpu-operator              v1.8.1          v1.8.1          NVIDIA GPU Operator creates/configures/manages ...
nvaie/gpu-operator-1-1          v1.9.1          v1.9.1          NVIDIA GPU Operator creates/configures/manages ...
nvaie/gpu-operator-1-2          v1.10.1         v1.10.1         NVIDIA GPU Operator creates/configures/manages ...
nvaie/gpu-operator-2-0          v1.10.1         v1.10.1         NVIDIA GPU Operator creates/configures/manages ...
nvaie/gpu-operator-2-1          v1.11.0         v1.11.0         NVIDIA GPU Operator creates/configures/manages ...
nvaie/gpu-operator-2-2          v1.11.1         v1.11.1         NVIDIA GPU Operator creates/configures/manages ...
nvaie/network-operator-1-1      1.1.0           v1.1.0          Nvidia network operator
nvaie/network-operator-1-2      1.1.0           v1.1.0          Nvidia network operator
nvaie/network-operator-2-0      1.1.0           v1.1.0          Nvidia network operator
nvaie/network-operator-2-1      1.2.0           v1.2.0          Nvidia network operator
nvaie/network-operator-2-2      1.2.0           v1.2.0          Nvidia network operator

```

ここはおとなしく[公式手順](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/install-gpu-operator-nvaie.html#installing-gpu-operator)通り、Helmで最新のGPU-Operatorの*gpu-operator-2-1*をデプロイします。

```bash
$ helm install --wait gpu-operator nvaie/gpu-operator-2-1 -n gpu-operator
NAME: gpu-operator
LAST DEPLOYED: Fri Sep  9 08:22:09 2022
NAMESPACE: gpu-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

少し待ってから、PODの状態を確認してみます。

```bash
$ k get pod -w -n gpu-operator
NAME                                                          READY   STATUS      RESTARTS   AGE
gpu-feature-discovery-h96zt                                   1/1     Running     0          3m50s
gpu-operator-6dcddf646c-z7hlw                                 1/1     Running     0          4m32s
gpu-operator-node-feature-discovery-master-74bb868fb9-tvlxz   1/1     Running     0          4m32s
gpu-operator-node-feature-discovery-worker-64wnh              1/1     Running     0          4m32s
gpu-operator-node-feature-discovery-worker-gfpcp              1/1     Running     0          4m32s
gpu-operator-node-feature-discovery-worker-nzbbn              1/1     Running     0          4m32s
nvidia-container-toolkit-daemonset-6wdh7                      1/1     Running     0          3m50s
nvidia-cuda-validator-jphtj                                   0/1     Completed   0          58s
nvidia-dcgm-exporter-qjkjn                                    1/1     Running     0          3m50s
nvidia-device-plugin-daemonset-4g478                          1/1     Running     0          3m50s
nvidia-device-plugin-validator-n9bxf                          0/1     Completed   0          12s
nvidia-driver-daemonset-l49fb                                 1/1     Running     0          3m50s
nvidia-mig-manager-xdcdc                                      1/1     Running     0          32s
nvidia-operator-validator-sz8cd                               1/1     Running     0          3m38s
```

いい感じにPodが起動しています。

ちなみにクラスタ作成時に*/var/lib/containerd*に少しボリュームを渡した理由は、Disk Pressureが原因でPodがEvictしてしまう場合があるからです。

```
$ k get pod
NAME                                                          READY   STATUS     RESTARTS   AGE
gpu-feature-discovery-twb44                                   1/1     Running    0          8m40s
gpu-operator-6dcddf646c-b7lfw                                 1/1     Running    0          9m16s
gpu-operator-node-feature-discovery-master-74bb868fb9-hz5kq   1/1     Running    0          9m16s
gpu-operator-node-feature-discovery-worker-49fhf              1/1     Running    0          9m16s
gpu-operator-node-feature-discovery-worker-sq4kr              1/1     Running    0          9m16s
gpu-operator-node-feature-discovery-worker-sqj77              1/1     Running    0          9m16s
nvidia-container-toolkit-daemonset-jkhmp                      1/1     Running    0          8m40s
nvidia-cuda-validator-rzkt5                                   0/1     Evicted    0          3m17s
nvidia-dcgm-exporter-wbdhz                                    1/1     Running    0          8m40s
nvidia-device-plugin-daemonset-ppg9s                          1/1     Running    0          8m40s
nvidia-driver-daemonset-kt5pb                                 1/1     Running    0          8m40s
nvidia-mig-manager-f8qb4                                      1/1     Running    0          8m40s
nvidia-operator-validator-xmbgj                               0/1     Init:2/4   1          8m25s

$ k describe pod nvidia-cuda-validator-rzkt5
Name:           nvidia-cuda-validator-rzkt5
Namespace:      gpu-operator
Priority:       0
Node:           tkg-gpu-cluster01-tkg-gpu-cluster01-nodepool-02-kjbcg-6b749hk6h/
Start Time:     Fri, 09 Sep 2022 08:48:29 +0900
Labels:         app=nvidia-cuda-validator
Annotations:    kubernetes.io/psp: gpu-operator-privileged
Status:         Failed
Reason:         Evicted
Message:        Pod The node had condition: [DiskPressure].
IP:
IPs:            <none>
Controlled By:  ClusterPolicy/cluster-policy
Init Containers:
  cuda-validation:
    Image:      nvcr.io/nvidia/cloud-native/gpu-operator-validator:v1.11.0
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
    Args:
      vectorAdd
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from nvidia-operator-validator-token-6mr6r (ro)
Containers:
  nvidia-cuda-validator:
    Image:      nvcr.io/nvidia/cloud-native/gpu-operator-validator:v1.11.0
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
    Args:
      echo cuda workload validation is successful
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from nvidia-operator-validator-token-6mr6r (ro)
Volumes:
  nvidia-operator-validator-token-6mr6r:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  nvidia-operator-validator-token-6mr6r
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
                 nvidia.com/gpu:NoSchedule op=Exists
Events:
  Type     Reason   Age    From     Message
  ----     ------   ----   ----     -------
  Warning  Evicted  3m49s  kubelet  The node had condition: [DiskPressure].
```

あと、Nvidia AI EnterpriseのHelmチャートでなければうまくGPU Operator動かなかったです。オープンのものと仕様が違う模様です。

### 動作確認
GPUをアサインしたPodで*nvidia-smi*コマンドを発行してみます。

```bash
$ kubectl run -it \
>     -n gpu-operator \
>     --limits=nvidia.com/gpu=1 \
>     --restart=Never \
>     --image=nvcr.io/nvidia/cuda:10.2-devel-ubuntu18.04 \
>     --overrides='{ "spec": { "template": { "spec": { "imagePullSecrets": [{"name": "ngc-secret"}] } } } }' \
>     test -- nvidia-smi

$ k get pod
NAME                                                          READY   STATUS      RESTARTS   AGE
gpu-feature-discovery-ddv2m                                   1/1     Running     0          6m7s
gpu-operator-6dcddf646c-58bn5                                 1/1     Running     0          6m48s
gpu-operator-node-feature-discovery-master-74bb868fb9-jgxw6   1/1     Running     0          6m48s
gpu-operator-node-feature-discovery-worker-664wl              1/1     Running     0          6m48s
gpu-operator-node-feature-discovery-worker-k9vqx              1/1     Running     0          6m48s
gpu-operator-node-feature-discovery-worker-kjq94              1/1     Running     0          6m48s
nvidia-container-toolkit-daemonset-7j5xz                      1/1     Running     0          6m8s
nvidia-cuda-validator-jnpbq                                   0/1     Completed   0          2m51s
nvidia-dcgm-exporter-s6t7l                                    1/1     Running     0          6m7s
nvidia-device-plugin-daemonset-54fb2                          1/1     Running     0          6m7s
nvidia-device-plugin-validator-9pw8n                          0/1     Completed   0          2m34s
nvidia-driver-daemonset-5l9b7                                 1/1     Running     0          6m8s
nvidia-mig-manager-bg6jn                                      1/1     Running     0          2m6s
nvidia-operator-validator-l4rmd                               1/1     Running     0          5m51s
test                                                          0/1     Completed   0          73s

$ k logs test
Fri Sep  9 05:56:21 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.73.08    Driver Version: 510.73.08    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  GRID A100-2-10C     On   | 00000000:02:00.0 Off |                   On |
| N/A   N/A    P0    N/A /  N/A |                  N/A |     N/A      Default |
|                               |                      |              Enabled |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| MIG devices:                                                                |
+------------------+----------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |         Memory-Usage |        Vol|         Shared        |
|      ID  ID  Dev |           BAR1-Usage | SM     Unc| CE  ENC  DEC  OFA  JPG|
|                  |                      |        ECC|                       |
|==================+======================+===========+=======================|
|  0    0   0   0  |      0MiB /  9195MiB | 28      0 |  2   0    1    0    0 |
|                  |      0MiB /  4096MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```

無事、GPUがPodから見れました。片付けします。

```bash
$ k delete po test -n gpu-operator
pod "test" deleted

```
