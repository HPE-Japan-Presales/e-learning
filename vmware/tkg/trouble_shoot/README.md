# トラブルシュート系
検証等で遭遇したトラブルをまとめています。参考にしてください。

## vSphere再起動後にマネージメントクラスタにアクセスできない
コンテキストをマネージメントクラスターに変更して、ノード情報を取得してみます。

```
>kubectl config get-contexts
CURRENT   NAME                                                                    CLUSTER                           AUTHINFO                                NAMESPACE
*         takluster01-admin@takluster01                                           takluster01                       takluster01-admin
          tkg-mgmt-vsphere-20210830120158-admin@tkg-mgmt-vsphere-20210830120158   tkg-mgmt-vsphere-20210830120158   tkg-mgmt-vsphere-20210830120158-admin
 
>kubectl config use-context tkg-mgmt-vsphere-20210830120158-admin@tkg-mgmt-vsphere-20210830120158
Switched to context "tkg-mgmt-vsphere-20210830120158-admin@tkg-mgmt-vsphere-20210830120158".

>kubectl config view -o=jsonpath='{.current-context}'
'tkg-mgmt-vsphere-20210830120158-admin@tkg-mgmt-vsphere-20210830120158'

>kubectl get node
Unable to connect to the server: dial tcp 192.168.3.70:6443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.

```

マネージメントクラスターのkubeapiにアクセスできませんでした。VIPは192.168.3.70です。マネージメントクラスタはVIPをkube-vipを使って提供していると思うので、kube-vipのログを確認するために直接マスターノードにログインしてみます。マネージメントクラスターのマスターノードは**capv**ユーザーと、クラスター作成時に作成したssh keyでログインできる様です。

ssh接続後、*crictl*コマンドを使ってkube-vipの確認をしてみます。

```
$ crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
b29022909bd4a       05d7f1f146f50       17 hours ago        Running             kube-vip                  2                   ea55faaabff82
f7a1f6c96d926       34cd6462aea3a       17 hours ago        Running             kube-scheduler            2                   4cc34f85b0cbb
a6103199de7e5       7e8a07f21234b       17 hours ago        Running             kube-controller-manager   3                   d1130d52a35e3

$ crictl logs b29022909bd4a
E1119 01:00:16.774342       1 leaderelection.go:322] error retrieving resource lock kube-system/plndr-cp-lock: Get "https://tkg-mgmt-vsphere-20210830120158-control-plane-6ssmn:6443/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/plndr-cp-lock": dial tcp 127.0.0.1:6443: connect: connection refused
E1119 01:00:20.263602       1 leaderelection.go:322] error retrieving resource lock kube-system/plndr-cp-lock: Get "https://tkg-mgmt-vsphere-20210830120158-control-plane-6ssmn:6443/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/plndr-cp-lock": dial tcp 127.0.0.1:6443: connect: connection refused

$ curl -k https://127.0.0.1:6443
curl: (7) Failed to connect to 127.0.0.1 port 6443: Connection refused

```

何やらkube-vipコンテナがkube-apiサーバーにアクセスできていない様です:(

```
$ ss -atn
State                    Recv-Q                  Send-Q                                   Local Address:Port                                    Peer Address:Port                  Process
LISTEN                   0                       4096                                           0.0.0.0:111                                          0.0.0.0:*
LISTEN                   0                       4096                                         127.0.0.1:10257                                        0.0.0.0:*
LISTEN                   0                       4096                                         127.0.0.1:10259                                        0.0.0.0:*
LISTEN                   0                       4096                                     127.0.0.53%lo:53                                           0.0.0.0:*
LISTEN                   0                       128                                            0.0.0.0:22                                           0.0.0.0:*
LISTEN                   0                       4096                                         127.0.0.1:39335                                        0.0.0.0:*
LISTEN                   0                       4096                                         127.0.0.1:10248                                        0.0.0.0:*
SYN-SENT                 0                       1                                         192.168.3.53:42504                                   192.168.3.70:6443
ESTAB                    0                       0                                         192.168.3.53:22                                      192.168.3.31:50277
LISTEN                   0                       4096                                                 *:10250                                              *:*
LISTEN                   0                       4096                                              [::]:111                                             [::]:*
LISTEN                   0                       128                                               [::]:22                                              [::]:*

$ crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                               ATTEMPT             POD ID
8e87fe156dd1d       2adbfd9903bc3       2 minutes ago       Exited              etcd                               225                 5775b44308966
38d747fc30da0       52591a96c7c73       3 minutes ago       Exited              kube-apiserver                     1                   4a1f955737ddb
b29022909bd4a       05d7f1f146f50       18 hours ago        Running             kube-vip                           2                   ea55faaabff82
f7a1f6c96d926       34cd6462aea3a       18 hours ago        Running             kube-scheduler                     2                   4cc34f85b0cbb
a6103199de7e5       7e8a07f21234b       18 hours ago        Running             kube-controller-manager            3                   d1130d52a35e3
6975bdff0bf5f       05d7f1f146f50       18 hours ago        Exited              kube-vip                           1                   64ddcaf21555d
0ab46135f1a4c       34cd6462aea3a       18 hours ago        Exited              kube-scheduler                     1                   67b8abf130ffa
964365cdfa61c       7e8a07f21234b       18 hours ago        Exited              kube-controller-manager            2                   d5ab1a08fdaeb
d7d2e0c50536c       baa0fb8b14b2e       2 months ago        Exited              liveness-probe                     0                   f420ab45c5d4b
6f5d00e3e9b16       e563638a5b5d0       2 months ago        Exited              vsphere-cloud-controller-manager   1                   670580f02cf96
042b25bfcaca6       6789024834e4b       2 months ago        Exited              vsphere-csi-node                   0                   f420ab45c5d4b
45c7d9d5e0832       c86bd1406bf3d       2 months ago        Exited              antrea-ovs                         0                   cf62146d49783
a9b3ad8344dcd       c86bd1406bf3d       2 months ago        Exited              antrea-agent                       0                   cf62146d49783
367b868303c2e       c86bd1406bf3d       2 months ago        Exited              install-cni                        0                   cf62146d49783
2ec57ba58bd28       de7fbddd52db0       2 months ago        Exited              node-driver-registrar              0                   f420ab45c5d4b
36a19c9190837       84cf19a9fcb9c       2 months ago        Exited              manager                            0                   bd0c3cc48b429
23b3060d94d5f       5ba75fed2465d       2 months ago        Exited              manager                            0                   fc72bb0c3da20
d56e6a4b3b7a2       2e9c528bc4412       2 months ago        Exited              kube-rbac-proxy                    0                   bd0c3cc48b429
c504c05138558       2e9c528bc4412       2 months ago        Exited              kube-rbac-proxy                    0                   fc72bb0c3da20
c754d64e2f8ef       3170b86358df5       2 months ago        Exited              kube-proxy                         0                   9caacb33b142a

```

色々なコンテナが起動失敗してます。たぶんetcdが起動してないからkube-apiも起動できないんではないかと思います。ログを見てみましょう。

```
$ crictl logs 8e87fe156dd1d
[WARNING] Deprecated '--logger=capnslog' flag is set; use '--logger=zap' flag instead
2021-11-19 01:55:33.674396 I | etcdmain: etcd Version: 3.4.13
2021-11-19 01:55:33.674592 I | etcdmain: Git SHA: GitNotFound
2021-11-19 01:55:33.674630 I | etcdmain: Go Version: go1.15.8
2021-11-19 01:55:33.674699 I | etcdmain: Go OS/Arch: linux/amd64
2021-11-19 01:55:33.674714 I | etcdmain: setting maximum number of CPUs to 2, total number of available CPUs is 2
2021-11-19 01:55:33.675045 N | etcdmain: the server is already initialized as member before, starting as etcd member...
[WARNING] Deprecated '--logger=capnslog' flag is set; use '--logger=zap' flag instead
2021-11-19 01:55:33.675264 I | embed: peerTLS: cert = /etc/kubernetes/pki/etcd/peer.crt, key = /etc/kubernetes/pki/etcd/peer.key, trusted-ca = /etc/kubernetes/pki/etcd/ca.crt, client-cert-auth = true, crl-file =
2021-11-19 01:55:33.676068 C | etcdmain: listen tcp 192.168.3.54:2380: bind: cannot assign requested address

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a4:c1:91 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.53/24 brd 192.168.3.255 scope global dynamic eth0
       valid_lft 618869sec preferred_lft 618869sec
    inet6 fe80::250:56ff:fea4:c191/64 scope link
       valid_lft forever preferred_lft forever

```

192.168.3.53のIPを持っているノードなのに、ETCDはなぜか*192.168.3.54*のIPを持とうとしています。
こちらの[ドキュメント](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.3/vmware-tanzu-kubernetes-grid-13/GUID-mgmt-clusters-vsphere.html)見ると、マネージメントクラスターをデプロイ後、DHCPサーバーに対してマネージメントクラスターのIPアドレスをDHCPサーバー側で予約しなきゃいけない様に読み取れます。確かに、そうじゃないですと再起動後に各ノードのIP変わっちゃいますね。

etcdのマニフェストで当初アサインされていたIPアドレスを確認します。もしくはマニフェストのIPを変えてもいいです。

```
$ cat /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.3.54:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.3.54:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.3.54:2380
    - --initial-cluster=tkg-mgmt-vsphere-20210830120158-control-plane-5clpw=https://192.168.3.51:2380,tkg-mgmt-vsphere-20210830120158-control-plane-2zqj7=https://192.168.3.53:2380,tkg-mgmt-vsphere-20210830120158-control-plane-6ssmn=https://192.168.3.54:2380
    - --initial-cluster-state=existing
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.3.54:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.3.54:2380
    - --name=tkg-mgmt-vsphere-20210830120158-control-plane-6ssmn
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

```
上記の例だと192.168.3.54がクラスタがデプロイされた際に持っていたIPです。DHCPサーバー側でIPアドレスが予約できましたらVMをリブートします。その後、kubectlとtanzuコマンドを打ってみます。

```
$ kubectl get node
NAME                                                    STATUS   ROLES                  AGE   VERSION
tkg-mgmt-vsphere-20210830120158-control-plane-2zqj7     Ready    control-plane,master   81d   v1.20.5+vmware.2
tkg-mgmt-vsphere-20210830120158-control-plane-5clpw     Ready    control-plane,master   81d   v1.20.5+vmware.2
tkg-mgmt-vsphere-20210830120158-control-plane-6ssmn     Ready    control-plane,master   81d   v1.20.5+vmware.2
tkg-mgmt-vsphere-20210830120158-md-0-8558d5fbb5-hc6xs   Ready    <none>                 81d   v1.20.5+vmware.2

$ tanzu cluster list
  NAME         NAMESPACE  STATUS         CONTROLPLANE  WORKERS  KUBERNETES        ROLES   PLAN
  takluster01  default    createStalled  0/3           0/3      v1.20.5+vmware.2  <none>  prod
```
クラスタを作った時はDHCPへのIP予約を忘れない様にしましょう・・・

