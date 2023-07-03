# Lustre File System用CSIドライバーをインストールしてみる


## はじめに
あるお客様でLustre File SystemをEzmeral K8sのPV領域として使用したいという相談がありましたので、そのときの手順を備忘録として記載しています。

DDN社が公開している[Exa CSI Driver](https://github.com/DDNStorage/exa-csi-driver)を使用します。CSIドライバーがサポートしているk8sディストリビューションはVanilla K8sのように読めます。Ezmeral k8sはVanilla K8sなので問題なく動くことを期待します。

## 要件
- Ezmeral Runtimeが既にあること (本検証では5.3を使用)
- Lustre File systemがあること

## インストール手順
### Ezmeral Addmission Webhookの条件変更
使用する[Exa CSI Driver](https://github.com/DDNStorage/exa-csi-driver)はホストノードのルートディレクトリを[CSI Controller](https://github.com/DDNStorage/exa-csi-driver/blob/master/deploy/kubernetes/exascaler-csi-file-driver.yaml#L271)と[CSI Driver](https://github.com/DDNStorage/exa-csi-driver/blob/master/deploy/kubernetes/exascaler-csi-file-driver.yaml#L395)にマウントする仕組みになっています。Ezmeral k8sはセキュリティの理由から、ホストノードのルートディレクトリをPodにマウントすることを禁止しています。これはMutating Webhook Configurationsの**hpecp-webhook**がリクエストを審査して禁止しています。  
このままだと、CSI Driverが要求するホストノードのルートディレクトリをマウントできないなので、CSI Driverをインストールする**Namespace**だけこのAddmission Webhookを回避できるようにします。特定の**Label**がついた**Namespace**だけ、このAddmission Webhookを通さないようにします。  
今回は**lustre-csi**というラベルキーがあるものは**hpecp-webhook**を通さないようにします。以下の例に従って**hpecp-webhook**を変更してください。

```
$ kubectl edit mutatingwebhookconfigurations hpecp-webhook
  name: soft-validate.hpecp.hpe.com
  namespaceSelector:			#変更
    matchExpressions:			#変更
    - key: lustre-csi				#変更
      operator: DoesNotExist		#変更
```

### Namespaceの作成
次に[Exa CSI Driver](https://github.com/DDNStorage/exa-csi-driver)をデプロイするための**Namespace**を作成します。先ほど指定した**Label**を忘れずにつけます。

```
$ kubectl create ns lustre-csi
namespace/lustre-csi created
 
$ kubectl label ns lustre-csi lustre-csi=true
namespace/lustre-csi labeled
 
$ kubectl get ns lustre-csi --show-labels
NAME                      STATUS   AGE     LABELS
lustre-csi                Active   73s     lustre-csi=true
```

### テスト
それでは本当にホストノードのルートディレクトリがマウントできるかテストしてみます。[こちら](manifest/test.yaml)のマニフェストを使って**lustre-csi**ネームスペースにPodできるか確認してみます。

```
$ kubectl apply -f test.yaml
pod/test created

$ kubectl get pod -n lustre-csi
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          7s

$ kubectl -n lustre-csi exec -it test -- ls /host
bin    dev    home   lib64  mnt    proc   run    srv    tmp    var
boot   etc    lib    media  opt    root   sbin   sys    usr
```
ホストノードのルートディレクトリがマウントされていることがわかります。

### CSI Driverのデプロイ
あとはDDN社の[手順](https://github.com/DDNStorage/exa-csi-driver)に従ってCSI Driverをデプロイしてください。その際、注意してほしいのは**Namespaced Resource**は全てnamespace: lustre-csiを指定してください。

お客様からはこの手順で正常に動いたというご連絡を頂きました。私の環境にLustre環境がないので、Storage Classがちゃんとできたか、PVCができたかなどのログが載せられなくて恐縮です:(