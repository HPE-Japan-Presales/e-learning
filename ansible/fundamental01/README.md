# Ansible基礎編01
## 対象者
- Ansibleを全く触ったことない方

## 事前準備
- Linux系サーバーx2台 (本セクションではUbuntu18.04とCentOS7.5を使っています)
  - 1台はAnsibleサーバーに、もう１台はAnsibleから操作する自動化対象サーバーにします
  - インターネットへの接続が可能であることを確認してください
  - rootユーザーでパスワード認証によるsshログイン可能なこと
- JSON構造の理解 

## はじめに
AnsibleはInfrastructure as Codeの技術を使ってIT自動化を実現するための構成管理ツールとなります。  
Codeといっても所謂*プログラミング言語*的なものではなく、シンプルで読みやすいYAML形式で記載できるため、プログラミングの経験がないような方でも始めやすいのがAnsibleの特徴です。  
また、Ansibleは膨大な**モジュール**というものが用意されています。モジュールは様々なアクションを簡単にコード化する謂わば*道具*になります。たとえば、Linux OS x1000台に対して*Tak*というユーザーを作るアクションを自動化する場合、*[user](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)*モジュールを使うことで**わずか数行のコード**として表せます。1000台分の紙の手順書(悪習？)と時間内に作業を終わらせるための人手はいりません。この**数行のコード**さえあればAnsibleが機械的に並列で一気に作業を行ってくれます。作業者は1人で十分で、コーヒーを飲んで待つだけです。  
ここで、「すでにユーザーがいる場合はどうするか？」「UIDが被ったときはどうするか」等々を気になった方がいらっしゃると思いますが、そのハンドリングはコード内で行う必要はありません。なぜなら、**モジュール**がそこらへんのことを全て行ってくれるからです。コード作成者は単に「Takユーザーを作成する」ということをシンプルにコード化すれば良いところが構成管理ツールの良いところです。(もし既にユーザーが存在する場合は...といったハンドリングももちろんできます。)  

百聞は一見にしかずなので、エンジニアならとりあえず手を動かして体感してみましょう！  

## Ansibleのインストール
Ansibleを動かすためにはPythonが必要です。メジャーなLinux distributionではPythonがデフォルトでインストールされています。Pythonがインストールされているか事前に用意したAnsible用Linuxにログインして確認してみましょう。

```
[root@ansible-learning-centos ~]# which python
/usr/bin/python
[root@ansible-learning-centos ~]# which python2
/usr/bin/python2
[root@ansible-learning-centos ~]# which python3
/usr/bin/which: no python3 in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
```
例ではCentOS環境のPythonを確認したところ、Python2がインストール済みであることがわかりました。Ubuntu環境ではPython3が入っていますが、*/usr/bin/python*のパスは存在していないと思います。Ansibleをインストールする際にPython2と*/usr/bin/python*のパスが作られると思うので無視で大丈夫です。  
それではAnsibleをインストールしてみます。  

**CentOS**  

```
[root@ansible-learning-centos ~]# yum install -y epel-release
~~~
Installed:
  epel-release.noarch 0:7-11                                                                                                                                                             

Complete!

[root@ansible-learning-centos ~]# yum install -y ansible
~~~
Installed:
  ansible.noarch 0:2.9.24-2.el7                                                                                                                                                          

Dependency Installed:
  PyYAML.x86_64 0:3.10-11.el7                                  libyaml.x86_64 0:0.1.4-11.el7_0          python-babel.noarch 0:0.9.6-8.el7        python-backports.x86_64 0:1.0-8.el7   
  python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7   python-cffi.x86_64 0:1.6.0-5.el7         python-enum34.noarch 0:1.0.4-1.el7       python-idna.noarch 0:2.4-1.el7        
  python-ipaddress.noarch 0:1.0.16-2.el7                       python-jinja2.noarch 0:2.7.2-4.el7       python-markupsafe.x86_64 0:0.11-10.el7   python-paramiko.noarch 0:2.1.1-9.el7  
  python-ply.noarch 0:3.4-11.el7                               python-pycparser.noarch 0:2.14-1.el7     python-setuptools.noarch 0:0.9.8-7.el7   python-six.noarch 0:1.9.0-2.el7       
  python2-cryptography.x86_64 0:1.7.2-2.el7                    python2-httplib2.noarch 0:0.18.1-3.el7   python2-jmespath.noarch 0:0.9.4-2.el7    python2-pyasn1.noarch 0:0.1.9-7.el7   
  sshpass.x86_64 0:1.06-2.el7                                 

Complete!

[root@ansible-learning-centos ~]# ansible --version
ansible 2.9.24
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```

**Ubuntu**

```
root@ansible-learning-ubuntu:~# apt update
~~~
Reading package lists... Done
Building dependency tree       
Reading state information... Done
96 packages can be upgraded. Run 'apt list --upgradable' to see them.

root@ansible-learning-ubuntu:~# apt install -y ansible
~~~

root@ansible-learning-ubuntu:~# which python
/usr/bin/python

root@ansible-learning-ubuntu:~# which python2
/usr/bin/python2

root@ansible-learning-ubuntu:~# ansible --version
ansible 2.5.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Feb 27 2021, 15:10:58) [GCC 7.5.0]
```

CentOSとUbuntuでAnsibleのバージョンは違いますが、無視して進めます。Ansibleが無事インストールされましたでしょうか？

## Yet Another Markup Language(YAML)について
YAMLはデータ構造を表現するための言語となります。人が見て直感的に読み描きできる構造が人気となっています。Kubernetes等含め、近年のソフトウェアではYAMLを使って設定内容を表現することが多いです。  
ここからはYAMLの描き方について見ていきます。

### Key-Value
key valueは所謂、連想配列やマップです。データをKeyとValueで表現させ、必要なデータをkeyを指定すること取得します。
KeyとValueの境には**:**と**スペース**を挿入します。
たとえば、朝ごはんに何を食べたかを取得できるようなデータ構造をYAMLで表すと以下のようになります。

```yaml
朝ごはん: パン
```
上の例では*朝ごはん*は*パン*を示していることになります。つまり、*朝ごはん*というkeyを指定すると*パン*という値が読み出せるようなデータ構造となっています。JSON形式で書くと以下のようになります。

```json
{
  "朝ごはん": "パン"
}
```

### リスト
リストは所謂、配列やスライスです。1つのkeyに対して複数のvalueが含まれるイメージです。  
リストは**-**と**スペース**を使って表現します。

```yaml
朝ごはん:
- パン
- 牛乳
- ヨーグルト
- フルーツ
```
上の例では*朝ごはん*というkeyを指定すると[パン、牛乳、ヨーグルト、フルーツ]全てが値として取得できます。JSONで表すと以下のようになります。

```json
{
  "朝ごはん": [
    "パン",
    "牛乳",
    "ヨーグルト",
    "フルーツ"
  ]
}
```


### ネスト構造
先に学んだKey-Valueやリストは*入れ子*にして使う場合があります。たとえば、*本日の食事*というKeyを指定した時、朝昼晩全てのメニューをValueとして取得する場合はどうすればいいでしょうか？  
このような少し複雑なデータ構造を表現するための方法が**ネスト**です。ネストを表現する上で重要になるのは**インデント**です。

#### インデント
インデント(行の開始位置)を使うことでデータネスト構造を表現できます。インデントが違うだけで違ったデータ構造として見えてしまいます。  
まずは*本日の食事*というkeyを入れた時にメインディッシュ(value)を取得できるようなデータ構造を目指します。

```yaml
本日の食事:
  朝ごはん: パン
  昼ごはん: ラーメン
  晩ごはん: 焼肉
```
朝、昼、晩ごはんのkeyの前にインデントが入っていることがわかると思います。
上記のYAMLをJSON形式にすると以下になります。

```json
{
  "本日の食事": {
    "朝ごはん": "パン",
    "昼ごはん": "ラーメン",
    "晩ごはん": "焼肉"
  }
}
```

上の例では*本日の食事*というkeyを指定すると、以下のデータが取得できます。

```yaml
朝ごはん: パン
昼ごはん: ラーメン
晩ごはん: 焼肉
```
このデータから更に朝ごはんや昼ごはんといったkeyを指定することでメイディッシュを取得できます。  
目的通り、朝、昼、晩ごはんのメインディッシュが取得できていることがわかります。このようにkey-value等のデータ構造を入れ子にすることを**ネスト**と呼び、さらにそれをYAML表現するための方法が**インデント**となります。  

先ほどのYAMLから試しにインデントを抜いてみます。

```yaml
本日の食事:
朝ごはん: パン
昼ごはん: ラーメン
晩ごはん: 焼肉
```
全ての行のkey-valueが同じ開始位置で始まっていることが分かります。これをJSONで表してみます。

```json
{
  "本日の食事": null,
  "朝ごはん": "パン",
  "昼ごはん": "ラーメン",
  "晩ごはん": "焼肉"
}
```
本日の食事というkeyに対する値がnull(空)になってしまったことがわかります。なぜならインデントを行わなかったためネスト構造ではなくなったためです。  
それではメインディッシュだけでなく、全てのメニューを取得できるよなデータ構造を考えます。ご想像通り、key-value、リスト、ネスト構造を駆使して表現します。

```yaml
本日の食事:
  朝ごはん: 
  - パン
  - 牛乳
  - ヨーグルト
  - フルーツ
  昼ごはん: 
  - ラーメン
  - チャーハン
  - 餃子
  晩ごはん: 
  - 焼肉
  - サラダ
  - キムチ
  - アイス
  - ビール
```
これをJSONで表すと以下のようになります。

```json
{
  "本日の食事": {
    "朝ごはん": [
      "パン",
      "牛乳",
      "ヨーグルト",
      "フルーツ"
    ],
    "昼ごはん": [
      "ラーメン",
      "チャーハン",
      "餃子"
    ],
    "晩ごはん": [
      "焼肉",
      "サラダ",
      "キムチ",
      "アイス",
      "ビール"
    ]
  }
}
```
本日の食事というkeyを指定してデータを取得すると、朝、昼、晩のメニューを取得できるようになりました。このようにネスト構造を使ってAnsibleに対するYAML(Infrastructure as Codeのコード)を記載します。慣れるまで時間がかかるかもしれませんが、自身でどんどんYAMLを描いていけばすぐに慣れてきます。

インデントを表現するためのスペース数ですが、2または4が一般的と思います。タブは使わないでください。

## Playbook
**Playbook**はAnsible君に実行させるアクションをまとめた**コード**になります。つまり、自動化させたい作業等をPlaybook化することで、作業者は寒いデータセンターの中で時間に追われて、くしゃくしゃになった紙の手順書にチェックをつけて１つ１つコマンドを実行する必要はなく、Ansibleを実行させた後はあったかい休憩室で待つだけで作業は終了します。

### Pingを打つだけのPlaybookを作ってみよう!
ここからはPlaybookを作成していきます。まずは事前に用意した*自動化対象サーバー*にpingを打つだけのPlaybookを作成します。
Playbookを作成する際は、自動化したい作業に使えるようなAnsibleモジュールを探すところから始めます。

#### モジュールを探す
Ansibleモジュールは大きく3つに分けれらます。  

- Core Module  
Ansibe開発チームによってメンテナンスされる高品質なモジュールとなります。RH

- Network Module
Ansible Networkチームによってメンテナンスされる高品質なモジュールとなります。

- Certified Module(認定モジュール)  
Ansible Partnerとして認定された企業、団体が作成する高品質なモジュールとなります。

- Community Module(コミュニティモジュール)  
コミュニティによって作成されるモジュールで、Ansible開発チームは関与しません。そのため、利用者はこのモジュールの品質を自身で評価する必要があります。

モジュール種別の詳細を知りたい方は以下を確認してください。

- [Module Maintenance & Support
](https://docs.ansible.com/ansible/2.9/user_guide/modules_support.html)

HPEはAnsible認定パートナーとなっており、Aruba NetworkとNimble Storage、HPE OneViewといったハードウェアレイヤの自動化に関わる認定モジュールをリリースしています。その他、[Ansible認定モジュールはこちら](https://access.redhat.com/articles/3642632)をご確認ください。

Ansible Core Moduleは以下のサイトで確認ができます。

- [Ansible Documentaion](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)

たとえば、[Ansible Documentaion](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)からブラウザの検索機能で*ping*と検索してみます。すると*[ping – Try to connect to host, verify a usable python and return pong on success](https://docs.ansible.com/ansible/2.9/modules/ping_module.html#ping-module)*という項目が見つかると思います。今回はこのモジュールを使います。  

また、認定モジュールや、コミュニティモジュールを探したい場合は[Ansible Galaxy](https://galaxy.ansible.com/)から探せます。Ansible Galaxyでは様々な人々がPlaybookも公開しているので、そのPlaybookをサンプルとして使うこともできます。

#### Playbookの構造
見つけた*[pingモジュール](https://docs.ansible.com/ansible/2.9/modules/ping_module.html#ping-module)*を使って、Playbookを描いてみましょう。Playbookは先ほど紹介したYAML形式で記載しますが、Ansibleに対して「どのサーバーに対してアクションを実行するか」「どういったアクションをするか」などを伝える必要があるため、Playbookを描くにあたって簡単なお作法がありますので、まずはそのお作法をみていきます。

その前に練習環境用のディレクトリを**Ansibleサーバー上**作りましょう。*playbook01*というディレクトリを作成してください。

```
[root@ansible-learning-centos ~]# mkdir playbook01
[root@ansible-learning-centos ~]# ls
anaconda-ks.cfg  playbook01
```

まずは「どのサーバーに対してアクションを実行するか」させるかを描きます。今回はAnsibleサーバーから自動化対象サーバーに対してPingをします。Ansibleで自動化対象を**Inventory**というものファイルに記載します。Inventoryには自動化対象サーバーの情報を記載します。  
それでは先ほど作成したディレクトリ*playbook01*ディレクトリに移動した後、更に*inventory*というディレクトリを作成します。

```
[root@ansible-learning-centos ~]# cd playbook01/
[root@ansible-learning-centos playbook01]# 
[root@ansible-learning-centos playbook01]# mkdir inventory
[root@ansible-learning-centos playbook01]# 
[root@ansible-learning-centos playbook01]# ls 
inventory
```

inventoryディレクトリを作成した後、実際に自動化対象のIPアドレスやホスト名を記載したInventoryファイルを作成します。ファイル名はなんでも良いのですが、今回は*scapegoats*というファイル名にします。ファイルを作成後、お好きなエディタでファイルを開いてください。

```
[root@ansible-learning-centos playbook01]# touch inventory/scapegoats
[root@ansible-learning-centos playbook01]# vi inventory/scapegoats 
```

自動化対象サーバー情報を記載していきます。inventoryファイルはYAML形式ではないので注意して下さい。

```
[scapegoats]
<Your Server IP>

[scapegoats:vars]
ansible_connection=ssh
ansible_ssh_user=root
ansible_ssh_pass=<ROOT PASSWORD>
```  

1行目の*\[scapegoats\]*の記載では*scapegoats*グループという自動化対象のグループを作っています。2行目にその*scapegoats*グループのメンバーとして今回用意した自動化対象サーバーのIPを記載しています。  
*[scapegoats:vars]*ではどうやって*scapegoats*グループに記載されたサーバーにアクセスするかを記載しています。**ansible\_connection**ではAnsibleからアクセス方法、**ansible\_ssh\_user**でsshでログインするユーザー名、**ansible\_ssh\_pass**ではsshでログインする際のパスワードを指定します。これらinventoryファイルに記載された認証情報を使ってAnsibleは自動化対象サーバーにアクセスします。



次にPlaybook本体を描いてきます。Playbook用のファイル*main.yaml*を作成した後、お好きなエディタで開いてください。

```
[root@ansible-learning-centos playbook01]# touch main.yaml
[root@ansible-learning-centos playbook01]# tree .
.
├── inventory
│   └── scapegoats
└── main.yaml

1 directory, 2 files
[root@ansible-learning-centos playbook01]# vi main.yaml
```

まず、「どのサーバーに対してアクションを実行するか」するかを描きます。PlaybookはYAML形式で記載します。今回はAnsibleサーバーから自動化対象サーバーに対してPingを発行します。先ほど作成したinventoryファイル内*scapegoats*グループをターゲットとして指定します。

```yaml:main.yaml
---
- hosts: scapegoats
```

*hosts*というKeyに対して、*scapegoats*を指定してリストとしてYAMLを描きました。つまり、この記載はAnsible君に対して「今回の作業対象はscapegoatsグループだよー」と教えています。一番先頭の**---**は「ここからYAMLをはじめるよー」という宣言になります。

JSON形式でこのYAMLを変換すると、

```json
[
  {
    "hosts": "scapegoats"
  }
]
```
このように表現します。リストの１要素がとして*host: scapegoats*がkey-valueとして存在しています。  


次に「どういったアクションをするか」を以下のように追記します。インデントに注意してください。

```yaml
---
- hosts: scapegoats
  tasks:
  - name: "pingします！"
    ping:
```

自動化したいアクションは*tasks*というKeyに対してリストとして記載します。*name*はタスクの名前を記載しています。今回の例ではpingするアクションを*pingします！*という名前(Value)を設定しています。*name*はご自身の好きなように記載してください。  
*ping*というKeyの記載は使用するモジュールを宣言しています。Valueは空です。なぜValueが空かというとpingモジュールの仕様だからです。そのモジュールがどういう仕様(宣言方法)なのかは先にお伝えした[Ansible Documentaion](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)等で１つ１つ確認する必要があります。  

JSON形式に直すと以下のようになります。

```json
[
  {
    "hosts": "scapegoats",
    "tasks": [
      {
        "name": "pingします！",
        "ping": null
      }
    ]
  }
]
```

では、main.yamlを保存して実行してみましょう！playbookの実行は***ansible-playbook*コマンドで実行できます。*-i*オプションはinventoryファイルを指定するオプションとなります。

```
[root@ansible-learning-centos playbook01]# ansible-playbook -i inventory/scapegoats main.yaml 

PLAY [scapegoats] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [172.16.11.102]

TASK [pingします！] *************************************************************************************************************************************************************************
ok: [172.16.11.102]

PLAY RECAP ******************************************************************************************************************************************************************************
172.16.11.102              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**PLAY RECAP**という最後の出力はAnsibleでタスク実行結果を示しています。172.16.11.102に対するアクションに関して、**ok=2**という出力になっていると思います。これは2つのタスクの実行結果がokであったことを示します。okという結果は「タスクは無事終わったよー」という意味になります。サーバー(自動化対象サーバー)の設定変更が発生した場合は**changed**という結果で示されます。たとえば、changed=1となっている場合は、１つのタスクでサーバーの設定変更を加えたということを示します。  

作成したPlaybookはpingを打つだけのタスクを１つ定義したのになぜ*ok=2*という2タスクの実行したような結果となったのでしょうか？*ansible-playbook*コマンドの出力を注意深くみると、*TASK \[Gathering Facts\]*というタスクが一番始めに勝手に実行されていることがわかります。Gather FactsはAnsibleがデフォルトで実行する*情報収集タスク*となり、**[setup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html)**モジュールを実際は実行しています。Ansibleは定義したタスクを実行する前に自動化対象サーバーの様々な情報収集をします。これが**構成管理ツール**と呼ばれる１つの所以です。  

どのような情報を収集しているか見てみましょう！Playbookを指定しないで単に*setup*モジュールのみを実行してみましょう。

```
[root@ansible-learning-centos playbook01]# ansible -i inventory/scapegoats  -m setup scapegoats
~~~
        "ansible_distribution": "Ubuntu", 
        "ansible_distribution_file_parsed": true, 
        "ansible_distribution_file_path": "/etc/os-release", 
        "ansible_distribution_file_variety": "Debian", 
        "ansible_distribution_major_version": "18", 
        "ansible_distribution_release": "bionic", 
        "ansible_distribution_version": "18.04", 
~~~
```
様々な情報がJSON形式で出力されたと思います。これはinventoryファイル内scapegoatsグループに入れたホスト情報になり、*TASK \[Gathering Facts\]*で行っていた情報収集となります。  
なぜこのような情報を最初に取得するのかというと、この情報をPlaybook内で使用するためです。たとえば、*Ubuntu*サーバーだけpingタス**したくない**場合はどうでしょうか？この情報を使えば自動化対象サーバーがどのようなOSであるかを事前に知らなくても簡単にそのような制御ができます。

実際にAnsibleがデフォルトで実行するGather Factsタスクからの情報を利用してPlaybookを描き直してみましょう。scapegoatsグループに記載したサーバーのOSディストリビューション**以外**はpingタスクを実行させることにします。期待する結果はGather Factsタスクのみ実行で*ok=1*となるでしょう。 私の環境ではscapegotsグループに入れた自動化対象サーバーはUbuntuなので、Ubuntu以外のOSの場合のみpingタスクを実行させるようにします。  
**when**という構文を使うことで、条件を指定できます。*ansible -i inventory/scapegoats  -m setup scapegoats*の実行結果から、*TASK \[Gathering Facts\]*で*"ansible_distribution": "Ubuntu"*という情報を取得していることがわかりました。この情報を使ってタスクの実行条件を以下のように指定あげます。自身の環境に合わせてOSディストリビューション名を設定してください。

```yaml
---
- hosts: scapegoats
  tasks:
  - name: "pingします！"
    ping:
    when: ansible_distribution != "Ubuntu"
```
**!=**は否定を表す方法になり、上の例だと*ansible\_distribution*が*Ubuntu*で**ないとき**を宣言しています。保存して再度Playbookを実行してあげます。

```
[root@ansible-learning-centos playbook01]# ansible-playbook -i inventory/scapegoats  main.yaml 

PLAY [scapegoats] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [172.16.11.102]

TASK [pingします！] *************************************************************************************************************************************************************************
skipping: [172.16.11.102]

PLAY RECAP ******************************************************************************************************************************************************************************
172.16.11.102              : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```

*ok=1*と*skipped=1*という結果になりました。**skipped**はタスクをスキップしたことを示します。今回の例ではAnsible君がpingタスクを実行しようとしましたが、タスクの実行条件に合うサーバーではなかったのでスキップしたことになります。  
そもそも、pingを実行するだけなのにinventoryファイルになぜ認証情報を入れるのか疑問に思われた方がいらっしゃったと思います。理由の１つはこのGather Factsタスクを実行するためにサーバーにログインしなければならなかったからです。  

もう１つの理由がpingモジュールの仕様として対象サーバーにログインしてPythonのpathを確認するという理由からです。pingモジュールがPythonのPathを確認するかの条件はGather Factsタスクが実行されたかどうかに左右されます。(詳しく書くと*discovered\_interpreter\_python*のkey-valueをAnsibleが持っているかどうか)  
Ansibleは自動化対象のサーバーにPythonがインストールされていることが、自動化対象サーバー側の１つ要件になります。(Linuxの場合、Windowsやネットワーク機器等は例外)  本当にPythonのPathを取得してくれているのか確認するために、Playbookを以下のように変更してください。

```yaml
---
- hosts: scapegoats
  gather_facts: false
  tasks:
  - name: "pingします！"
    ping:
    register: result

  - name: "pingモジュールの実行結果詳細"
    debug: 
      msg: "{{ result }}"
```
2行目で*gather\_facts: false*と指定して、Gather Factsタスクをあえて実行しないようにしています。  
**register**構文はモジュールの出力を保存するときに使う構文となり、Valueにその出力を呼び出すためのKey名を指定します。今回の例だと*result*というkeyを指定することでpingモジュールが出力した情報を後から参照できるようになります。  
次に「pingモジュールの実行結果詳細」という新しいタスクが増えていることがわかります。**[debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)**モジュールを使って、*result*のValueを画面出力させようとしています。  
*{{ result }}*という記載方法は、Ansibleの世界では変数(key)に格納されている値(value)を展開することを示します。(厳密にはJinja2の世界)  
では実行してみます。


```
[root@ansible-learning-centos playbook01]#  ansible-playbook -i inventory/scapegoats main.yaml 

PLAY [scapegoats] ***********************************************************************************************************************************************************************

TASK [pingします！] *************************************************************************************************************************************************************************
ok: [172.16.11.102]

TASK [pingモジュールの実行結果詳細] *****************************************************************************************************************************************************************
ok: [172.16.11.102] => {
    "msg": {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        }, 
        "changed": false, 
        "failed": false, 
        "ping": "pong"
    }
}

PLAY RECAP ******************************************************************************************************************************************************************************
172.16.11.102              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

私の環境の場合、自動化対象サーバーのPythonのPathが*"discovered\_interpreter\_python": "/usr/bin/python3"*と取得できていることがわかります。

### ユーザーを作成するPlaybookを作ってみよう!
次にユーザーを作成するPlaybookを作ります。毎度の作業となりますがいい感じのモジュールを[Ansible Documentaion](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)から探してみます。  
*[user – Manage user accounts](https://docs.ansible.com/ansible/2.9/modules/user_module.html#user-module)*というモジュールが発見できたと思います。これが今回ユーザー作成のために使うモジュールになります。  
Ansibleのモジュールは大量にあるので探すのが大変だと思われる方もいらっしゃるかと思います。その場合は"Ansible ユーザー作成"とかでググった方が早いかもしれません。(大量にモジュールがある点が他の構成管理ツールと比べて良いところなんですが...)

新しいディレクトリ*playbook02*を作成してPlaybookを作っていきます。inventoryファイルは*playbook01*ディレクトリからコピーしちゃいましょう。
```
[root@ansible-learning-centos playbook01]# cd
[root@ansible-learning-centos ~]# mkdir playbook02
[root@ansible-learning-centos ~]# cp -p playbook01/inventory/ playbook02/
cp: omitting directory ‘playbook01/inventory/’
[root@ansible-learning-centos ~]# cp -pr playbook01/inventory playbook02/
[root@ansible-learning-centos ~]# cd playbook02/
[root@ansible-learning-centos playbook02]# vi main.yaml
```

userモジュールを使ったPlaybookは以下となります。pingモジュールを使ったPlaybookとさほど変わらないことがわかると思います。作成するユーザー名はお好きな名前を指定していただいて構わないです。

```yaml
---
- hosts: scapegoats
  tasks:
    - name: "ユーザー作りまーす！"
      user:
        name: ansible-user
        state: present

```

上の例の場合、*ansible-user*というユーザーを作成しています。userモジュールではUser IDやGroup等も指定できます。詳しくは[userモジュール](https://docs.ansible.com/ansible/2.9/modules/user_module.html#user-module)の説明を見てください。  
ちなみにタスク名を日本語で記載していますが、個人的には全て英語で記載した方が良いと思います。なぜなら全角スペースがYAMLに含まれた場合にYAMLのデータ構造が崩れてエラーになってしまうからです。

では、実行してみます。

```
[root@ansible-learning-centos playbook02]# ansible-playbook -i inventory/scapegoats main.yaml 

PLAY [scapegoats] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [172.16.11.102]

TASK [ユーザー作りまーす！] ***********************************************************************************************************************************************************************
changed: [172.16.11.102]

PLAY RECAP ******************************************************************************************************************************************************************************
172.16.11.102              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

結果が*ok=2*で*changed=1*となりました。*ok=2*はGather Factsタスクとユーザー作成タスクが終わったことを示しており、*changed=1*は*ok*だったタスクのうち1つのタスクが自動化対象サーバーに対して設定変更を加えたことを示しています。今回の例だとユーザー作成タスクによりユーザーが作成されたことを示しています。実際にユーザーが作成されたかを対象のサーバーにログインして確認してみましょう。

```
[root@ansible-learning-centos playbook02]# ssh root@172.16.11.102
root@scapegoat02:~# 
root@scapegoat02:~# cat /etc/passwd
~~~
ansible-user:x:1001:1001::/home/ansible-user:/bin/sh
root@scapegoat02:~# exit

[root@ansible-learning-centos playbook02]# 
```
ちゃんとユーザーが作成されていることがわかります。

#### 冪等性(べきとうせい)
ユーザーは作成できましたが、もう１度Playbookを実行したらどうなるでしょうか？やってみましょう。

```
[root@ansible-learning-centos playbook02]# ansible-playbook -i inventory/scapegoats main.yaml 

PLAY [scapegoats] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [172.16.11.102]

TASK [ユーザー作りまーす！] ***********************************************************************************************************************************************************************
ok: [172.16.11.102]

PLAY RECAP ******************************************************************************************************************************************************************************
172.16.11.102              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

結果を見ると*ok=2 *で*changed=0*となりました。*ok=2*となっているのでタスクは終わっています。しかし、ユーザーは既に作成されているのでAnsible君から「もうユーザーいるから作れなかったよー」という感じのエラーが出力されると思われた方がいらっしゃるのではないでしょうか？  

Infrastructure as Codeの世界では**冪等性**が重要となります。冪等性は**何度やっても同じ状態になること**ということです。今回、user作成タスクをPlaybookに定義しましたが、それは「ユーザーを**作成すること**」というアクションを定義したのではなく、「ユーザーを**存在すること**」という**状態**を定義しています。なのでAnsible君は「ちゃんとansible-userは存在してましたよー」という確認をした上で何もせずに*ok=2*という結果を出しました。ただ、状況によっては**アクション**を定義することもあります。pingモジュールを使ったPlaybookは「pingを実行すること」というアクションになると考えます。Ansibleモジュールは冪等性を保障していますが、Playbookの作者自身も冪等性を意識してYAMLを記載しないと冪等性が崩れてしまう場合があるので注意してください。

#### 繰り返し(Loop)と変数
今、1人のユーザーを作成しましたが、ansible-user01~ansible-user05まで作成したい場合はどうしますか？以下の例のように同じようなタスクを羅列して記載しても良いですが可読性が悪いですし、メンテナンス性も悪くなります。数行で記載したほうがわかりやすく芸術的と考えます。

```yaml
---
- hosts: scapegoats
  tasks:
    - name: "ユーザー作りまーす！"
      user:
        name: ansible-user01
        state: present
    - name: "ユーザー作りまーす！"
      user:
        name: ansible-user02
        state: present
    - name: "ユーザー作りまーす！"
      user:
        name: ansible-user03
        state: present    
    - name: "ユーザー作りまーす！"
      user:
        name: ansible-user04
        state: present   
    - name: "ユーザー作りまーす！"
      user:
        name: ansible-user05
        state: present   
```

このような場合、変数と繰り返しの構文を使うと数行で芸術的に記載できます。以下のPlaybook例を見てください。

```yaml
---
- hosts: scapegoats
  vars:
    scapegoats_users:
    - ansible-user01
    - ansible-user02
    - ansible-user03
    - ansible-user04
    - ansible-user05
  tasks:
    - name: "ユーザー作りまーす！"
      user:
        name: "{{ item }}"
        state: present
      with_items: "{{ scapegoats_users }}"
```
3行目**vars**構文はPlaybook作者が変数を定義したい場合に使います。今回の例では*scapegoats\_users*というkeyに対してのvalueは作成したいユーザーのリストを指定しています。  
tasks内のユーザー作成タスク最終行を見てください。**with\_items**という構文を使っています。この構文は**与えられたリストの要素分タスクを実行する**構文になります。与えるリストは先ほど*vars*内に記載した*scapegoats\_users*というkeyがもつユーザーのリスト(value)です。先述の通り、Ansibleでは"{{ 変数名(Key名) }}"でその値(Value)が取得できるので、今回の例では\"{{ scapegoats\_users }}\"を指定しています。  
少し行を戻って、name: \"{{ item }}\"はwith\_itemsにセットされたリストの要素１つ１つが順番に入ります。  
つまり、このユーザー作成タスク１つでuserモジュールは５回実行されることになります。実際にやってみましょう。


```
[root@ansible-learning-centos playbook02]# ansible-playbook -i inventory/scapegoats main.yaml 

PLAY [scapegoats] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [172.16.11.102]

TASK [ユーザー作りまーす！] ***********************************************************************************************************************************************************************
changed: [172.16.11.102] => (item=ansible-user01)
changed: [172.16.11.102] => (item=ansible-user02)
changed: [172.16.11.102] => (item=ansible-user03)
changed: [172.16.11.102] => (item=ansible-user04)
changed: [172.16.11.102] => (item=ansible-user05)

PLAY RECAP ******************************************************************************************************************************************************************************
172.16.11.102              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

出力からansible-user01~ansible-user05まで作られたことがわかります。まだAnsible君を信用できない方は実際に自動化対象サーバーにログインして確かめてください。

## まとめ
本章ではAnsibleの基本的なことを記載しました。すでに感じていただいたと思いますが、Ansibleを使うことで簡単に作業は自動化できます。コマンドをあまり知らないジュニアなエンジニアの方でもAnsibleさえ学んでしまえば、コマンドは知っているけどansibleを知らないシニアなエンジニアの方よりも、同じ作業を早く確実にこなすことができます。  
私はAnsibleを「自ら考え行動はしてくれないけど、ちゃんと伝えれば確実に作業をこなしてくれて、不眠不休かつ無給で働いてくれるチームメイト」と思っています。なので君付けで記載していました。
ちなみにHPE製品を使えばハードウェアの設定も自動化でき、オンプレ環境の運用も楽々みたいです。  
ぜひ引き続きAnsibleを学んでいただければ幸いです。