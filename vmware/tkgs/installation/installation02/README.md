# vSphere with Tanzu / Tanzu K8s Grid Service Installation
NVAIEの基盤の下地を作るため、
vSphere with Tanzuの環境を作っていきます。

## TKGs構築② - 共有データストアの作成
次にクラスターを構成する各ホストがアクセス可能な共有データストアを用意します。<br>
シンプルに実装できるため今回はWindows ServerでNFS共有データストアを用意して実施してます。<br>
→<br>
実装途中で残念ながらWindows ServerのNFSではパフォーマンスが悪く、子クラスターを作成しする際にコンテナイメージをデプロイすることができませんでしたので、NFS利用時はLinuxで用意することをおすすめします。

![](pics/pic01.png)
今回はWindows Serverで用意

![](pics/pic02.png)

![](pics/pic03.png)
共有用のDドライブを用意し、<br>
Dドライブ＞”プロパティ”＞”詳細な共有”＞”このフォルダーを共有する”にチェックを入れて適用

![](pics/pic04.png)
サーバーマネージャーを開く

![](pics/pic05.png)
サーバーマネージャーより、 ”ファイルサービスと記憶域サービス” ＞ ”共有”
 ＞ ”タスク” ＞ ”新しい共有” ＞ ”プロファイルの選択” ＞ ”NFS共有-簡易” を選択

![](pics/pic06.png)
パスの設定

![](pics/pic07.png)
認証の設定

![](pics/pic08.png)
共有するESXiホストのアクセス許可を追加、vCenterサーバーは不要

![](pics/pic09.png)

![](pics/pic10.png)

![](pics/pic11.png)

![](pics/pic12.png)

![](pics/pic13.png)

![](pics/pic14.png)
Windows Server側の設定は完了

![](pics/pic15.png)
次に、vCenter側からESXiホストにマウントする

![](pics/pic16.png)
NFSバージョンの選択

![](pics/pic17.png)
データストアの名称とパスの指定

![](pics/pic18.png)
マウントするホストの選択

![](pics/pic19.png)

![](pics/pic20.png)
NFSデータストアが無事作成できました。
