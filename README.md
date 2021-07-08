[![Build Status](https://travis-ci.com/IBM/worklog.svg?branch=master)](https://travis-ci.com/IBM/worklog)

# 作業ログ(Work Log)

## 概要

Flask、MongoDB、Kubernetesを使用して「作業ログ(Work Log)」Webアプリケーションを作成します。
作業ログアプリケーションは、以下に示す作業者の日勤状況を記録・追記することができます。

* オフィス勤務
* テレワーク(リモートワーク)
* 休暇
* 休日
* 病欠

このリポジトリの内容を理解することで、以下に対する知識を得ることができます。

* Python Flask アプリケーションを作成する
* MongoDBをPythonアプリケーションに組み込む
* OpenShift上にマイクロサービスをデプロイし実行する

## アーキテクチャ

![architecture](readme_images/architecture.png)

アプリケーションは左から順に、3つの要素で構成されています。

1. Reactで構築されたUIWebアプリ
2. Python Flaskで構築された WebAPIアプリ
3. WebAPIアプリからアクセスされる、ユーザデータを保存するMongoDB

3つの要素は、以下のやり取りを行います。

1. ユーザは画面を操作して、アカウントの新規作成/ログイン/パスワードリセットを行うことができます。ユーザはログイン後、作業ログを表示、追加、編集できます。
2. UIは、Reactを通じて処理されます。 Reactは、API呼び出しが初期化される場所です。
3. API呼び出しは、KubernetesのFlask APIマイクロサービスで処理されます。
4. データは、API呼び出しを通してMongoDBに保存、または変更されます。
5. API呼び出しは、UIを通して処理されます。

## 含まれるコンポーネント

* [Red Hat OpenShift on IBM Cloud](https://www.ibm.com/jp-ja/cloud/openshift): IBMがフルマネージドで提供するRed Hat OpenShift on IBM Cloudは、IBM Cloudのエンタープライズ・クラスの規模とセキュリティーを活用して、更新、スケーリング、プロビジョニングを自動化します。
* [Swagger](https://swagger.io/): APIライフサイクル全体にわたる開発を可能にするOpenAPI仕様のAPI開発者ツールのフレームワークです。

<!--Update this section-->
## 特筆すべきテクノロジー

* [Microservices](https://www.ibm.com/developerworks/community/blogs/5things/entry/5_things_to_know_about_microservices): 軽量プロトコルを使用して、クラウド内の最新のアプリケーション構成でビルディングブロックを提供する、きめ細かい、緩く結合されたサービスの集合です。
* [Python](https://www.python.org/): Pythonは、より迅速に作業し、システムをより効果的に統合できるプログラミング言語です。
* [Flask](http://flask.pocoo.org/): WebAPIを構築するためのPythonのマイクロフレームワークです。
* [React](https://facebook.github.io/react/): ユーザインターフェイスを構築するためのJavaScriptライブラリです。
* [MongoDB](https://www.mongodb.com/): ドキュメント志向 NoSQLデータベースです。

# OpenShiftでの公開手順

アプリをOpenShift上で公開するには、次の手順を実施します。詳細は各項目で説明します。

1. [gitリポジトリのクローン](#1-gitリポジトリのクローン)
2. [ローカルでのアプリケーションの動作確認](#2-ローカルでのアプリケーションの動作確認)
3. [OpenShiftへのデプロイ](#3-OpenShiftへのデプロイ)

## 1. gitリポジトリのクローン

ローカル環境にこの `worklog` リポジトリをクローンします。ターミナルにて、以下を実行します。

```sh
git clone https://github.com/tty-kwn/worklog
cd worklog
```

## 2. ローカルでのアプリケーションの動作確認

ローカルで動作確認しない場合は、この手順は飛ばして構いません。

### 2.1. MongoDBのインストールと起動

MongoDBがインストールされていない場合は、以下を参照してインストールと起動を行ってください。

* windows: [Install MongoDB Community Edition on Windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
* macos: [Install MongoDB Community Edition on macOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)
* それ以外: [Install MongoDB](https://docs.mongodb.com/manual/installation/)

すでにインストール済みの場合は起動を行ってください。起動方法は上記インストール方法に記載されています。

### 2.2. Python Flaskで構築された WebAPIアプリの起動

以下を導入してください。

* [Python 3](https://www.python.org/downloads/)
* [Node.js(NPM)](https://nodejs.org/en/)

1. pythonのバージョンは2系ではなく3系を利用します。環境によってはpython/pipコマンドではなくpython3/pip3などである可能性もあるため留意してください。
2. pipコマンドを実行し、pythonライブラリをインストールします。

    ```sh
    pip install -r requirements.txt
    ```

3. app.shシェルスクリプトを実行し、Python Flaskアプリケーションを起動します。(もしpythonコマンドではなくpython3を利用する場合は、シェルスクリプト内部の`python`コマンドを`python3`コマンドに置き換えて実行してください！)

    ```sh
    ./app.sh
    ```

    windowsの場合はシェルスクリプトを利用せず、以下のコマンドを実行してください。

    ```sh
    python -m app.__init__
    ```

4. Flaskアプリが起動したら, Swagger APIドキュメント(`http://localhost:5000/api`)と [API.md](API.md) を使ってAPIの利用方法を理解することができます。

### 2.3. Reactで構築されたWebUIアプリの起動

1. Reactアプリディレクトリに移動します。`

    ```sh
    cd web/worklog
    ```

2. npmコマンドを実行し、nodejsライブラリをインストールします。

    ```sh
    npm install
    ```

3. Reactアプリを起動します。その際、Python FlaskサーバURLを環境変数REACT_APP_APP_SERVERに設定します(UIWebアプリ内部で利用しています)

    ```sh
    export REACT_APP_APP_SERVER=http://localhost:5000 && npm start
    ```

4. `http://localhost:3000`にアクセスしてください。React UIを確認することができます。

## 3. OpenShiftへのデプロイ

ここではOpenShift version4.6 のWebUI上でのデプロイ手順を示します。
CLIコマンド`oc`を用いた説明は行いません。

また、IBM Cloud上にRed Hat OpenShiftクラスタを作成済みであるものとします。
実際の操作は動画 [**openshift_operation.mp4**](openshift_operation.mp4) をダウンロードし確認してください。

### 3.1. プロジェクトの作成

UIをAdministratorモードにし、Projectを選択します。

「Create Project」ボタンをクリックしプロジェクト作成ダイアログにてプロジェクトを作成します。

ここでは、プロジェクト名は「worklog」とします。

### 3.2. 永続化ディスクとMongoDBの作成

deploy-mongodb.ymlにはMongoDBをコンテナとして公開するための設定が記述されています。

設定内容はPersistentVolumeClain(永続化ディスク)、StatementSet、Serviceの3つが記述されていますので、テキストエディタなどでファイルを開き、それぞれをコピーします。

#### 3.2.1. PersistentVolumeClaim(永続化ディスク)の作成

コンテナはデータを永続化しません。そのため、mongoDBのデータを永続化するためのディスク領域を定義します。永続化ディスクは「PersistentVolume」もしくは「PersistentVolumeClaim」で定義します。

詳細は[こちら](https://thinkit.co.jp/article/17470)をご参照ください。Kubernetesの記事ですが、OpenShiftでも考え方は全く同じです。

手順は以下のとおりです。

1. 事前に、deploy-mongodb.ymlから`kind: PersistentVolumeClaim`と記載された箇所をコピーしておきます。(`---`で囲まれた箇所です)
2. UIをAdministratorモードにし、Storage->Persistent Volume Claimsを選択します。
3. 「Create Persistent Volume Claim」ボタンをクリックし、「Create Persistent Volume Claim」画面を表示します。
4. 「Edit YAML」リンクをクリックし、deploy-mongodb.ymlからコピーしたテキストを貼り付けて「Create」ボタンをクリックします。

なお、ここで貼り付けたYAMLは、GUI上で操作する場合は以下の設定を行ったと同義です。

* Storage Class: ibmc-block-bronze
* Persistent Volume Claim Name: pvc
* Access Mode: Single User (RWO)
* Size: 2GiB

作成には数十秒を要します。しばらくお待ち下さい。

画面上部の「pvc」という名前の箇所の横に示された「pending」という砂時計アイコンが、「Bound」というアイコンに変われば完了です。

#### 3.2.2. StatefulSetの作成

続けて、StatefulSetを作成します。
StatefulSetは、PersistentVolumeClaimを使って安定した永続ストレージを供給する手段で、永続化したいデータを持つDBの作成などで利用されます。

手順は以下のとおりです。

1. 事前に、deploy-mongodb.ymlから`kind: StatefulSet`と記載された箇所をコピーしておきます。(`---`で囲まれた箇所です)
2. UIをAdministratorモードにし、Workloads->Stateful Setsを選択します。
3. 「Create Stateful Set」ボタンをクリックし、「Create Stateful Set」画面を表示します。
4. deploy-mongodb.ymlからコピーしたテキストを貼り付けて「Create」ボタンをクリックします。

これで、永続化ディスクを持つmongoDBが作成されました。

#### 3.2.3. Serviceの作成

続けて、上記で作成したStatefulSetをネットワークからアクセス可能するために、Serviceを作成します。
Serviceは、StatefulSetをローカルネットワーク上に公開し、同一ネットワーク内の他のアプリからアクセスできるようにする機構です。

手順は以下のとおりです。

1. 事前に、deploy-mongodb.ymlから`kind: Service`と記載された箇所をコピーしておきます。(`---`で囲まれた箇所です)
2. UIをAdministratorモードにし、Network->Servicesを選択します。
3. 「Create Service」ボタンをクリックし、「Create Service」画面を表示します。
4. deploy-mongodb.ymlからコピーしたテキストを貼り付けて「Create」ボタンをクリックします。

これで、mongoDBを他のアプリから参照可能となりました。

### 3.3. Python Flaskで構築された WebAPIアプリのs2iデプロイ

mongoDBを参照するPython Flask WebAPIアプリをデプロイします。

手順は以下のとおりです。

1. UIをDeveloperモードにし、StatefulSets「mongo」が表示されていることを確認します。
2. +Addを選択します。
3. 「From GIT」を選択します。
4. gitリポジトリURLを取得します。githubの場合は「code」ボタンをクリックし「HTTPS」タブに書かれたURLの右側にあるコピーボタンをクリックします。**注) HTTPSではなくSSH URLを選択する場合、この手順に加えてSecretの生成と登録が必要となります。**
5. 「Git Repo URL」に、コピーしたgitリポジトリURLを貼り付けます。
6. Builder Imageで「Python」を選択します。
7. 「Application Name」に「worklog」を、「Name」には「worklog-api」を入力します。
8. 「Routing」リンクをクリックします。「Target Port」に「5000」を入力し「Create 5000」を選択します。
9. 「Create」ボタンをクリックします。

これで、Python Flask WebAPIアプリのデプロイが開始されました。

(A)で書かれているのは「アプリケーション」を意図しており、Deploymentなどのオブジェクトをグルーピングする概念です。

先程の設定画面にて、「Application Name」で指定した「worklog」が表示されていることを確認することができます。
(なお、設定画面で「Name」で設定した値は、DeploymentのNameになります)

Kubernetesに慣れた方は、dockerリポジトリを通さずにソースコードを直接デプロイできることに驚かれたのではないでしょうか。これがOpenShiftの利点の1つ、s2i(source to image)機能です。

なお、ビルドの状況はTopology画面のBuildsに「Build #n is xxxx」というビルド状況と「View logs」というリンクがあるので、そこをクリックすることでビルドログを確認することができます。

同様に、ビルド完了後のアプリ起動確認はTopology画面のPodsに「View logs」というリンクがあるので、そこをクリックすることで動作ログを確認することができます。

### 3.4. Reactで構築された WebUIアプリのs2iデプロイ

WebAPIアプリを参照するWebUIアプリをデプロイします。

手順は以下のとおりです。

1. UIをDeveloperモードにし,Topologyを選択します。
2. WebAPIアプリ(worklog-api)をクリックし、Resourcesタブの最下部にあるRoutesから、Location URLをコピーします。コピーしたURLはテキストエディタなどに退避します(あとで使います)。
3. +Addを選択し、「From GIT」を選択します。
4. gitリポジトリURLを取得します。githubの場合は「code」ボタンをクリックし「HTTPS」タブに書かれたURLの右側にあるコピーボタンをクリックします。**注) HTTPSではなくSSH URLを選択する場合、この手順に加えてSecretの生成と登録が必要となります。**
5. 「Git Repo URL」に、コピーしたgitリポジトリURLを貼り付けます。
6. WebUIアプリはアプリケーションルートを(gitリポジトリルート)/web/worklogです。「Show Advanced Git Options」リンクをクリックし、表示された「Context Dir」に`/web/worklog`を指定します。
7. Builder Imageで「Nodejs」を選択します。
8. 「Application Name」に「worklog」を、「Name」には「worklog-ui」を入力します。
9. 「Routing」リンクをクリックします。「Target Port」に「3000」を入力し「Create 3000」を選択します。
10. 「Deployments」リンクをクリックします。ここでは環境変数を指定できます。このアプリでは内部でREACT_APP_APP_SERVERという変数をWebAPIサーバのURLとして利用しています。「NAME」には「REACT_APP_APP_SERVER」、「VALUE」には2でコピー、退避しておいたURLをペーストします。
11. 「Create」ボタンをクリックします。

これで、Reactで構築された WebUIアプリのデプロイが開始されました。
(動画では間違ってnginxを指定してしまっているのでいくつかやり直しています)

### 3.5. Ingressの作成

これまでの作業でもアプリケーションの公開は可能ですが、オプションとして、SSLオフロードなどのロードバランサの機能を提供するIngressをデプロイします。

1. IBM Cloudクラスタページに行き、画面を少し下にスライドしたところにある「ネットワーキング」の「Ingressサブドメイン」の文字列をコピーします。
1. ingress.ymlを開き、`<ingress subdomain here>`と記載している箇所に、コピーした「Ingressサブドメイン」を貼り付けます。その後、yamlに記載した内容をすべてコピーしておきます。
2. UIをAdministratorモードにし、Networking->Ingressesを選択します。
3. 「Create Ingress」ボタンをクリックし、「Create Ingress」画面を表示します。
4. ingress.ymlからコピーしたテキストを貼り付けて「Create」ボタンをクリックします。

以上で、Ingressの公開が完了しました。

### 3.6. 動作確認

`http://<INGRESS_SUBDOMAIN>` にアクセスすることで、WebUIページを、`http://<INGRESS_SUBDOMAIN>/api`にアクセスすることで、SwaggerAPIページを確認することができます。

注意) httpsの設定はしていませんので、httpでアクセスしてください。特にchromeは`http://`という文字列が隠れてしまうので、明示的に指定してください。

# CI/CDの設定と動作確認

openshiftでは、様々な方法でCI/CD(継続的インテグレーション、継続的デリバリー)を行う事ができます。

ここでは、github webhookを設定することで、簡単にCI/CDを実現できることを確認します。

具体的には、ソースコードを変更するだけで、テストやリリース工程で利用できるアプリケーションがビルドできること、アプリケーションがデプロイできることを確認します。

**注意)この作業ではgithubのリポジトリに管理者権限が必要です。よくわからない場合は管理者に相談してください。**

## 1. webhook設定

1. OpenShift Webコンソール画面にて、UIをDeveloperモードにし、Topologyを選択します。
2. 「worklog-ui」を選択し、ResourcesタブのBuilds下にある「BC worklog-ui」をクリックします。
3. BuildConfigページが表示されるので、画面一番下までスクロールします。
4. Webhooks URLが記載されています。一番下のType: Githubの右側にある「Copy URL with Secret」をクリックします。これでWebhooks URLがクリップボードにコピーされました。
5. githubのリポジトリページを開きます。Settingsタブをクリックします。(Settingsタブが無いという方は、そのリポジトリに管理者権限がありません。管理者に相談し管理者権限を付与してもらうか、管理者権限を持つ自身のアカウントにリポジトリをforkしてください)
6. Webhooksメニューを選択し、「Add webhook」ボタンをクリックします。もしパスワード入力画面が表示された場合は、パスワードを入力してください。
7. 「Payload URL」に先程コピーした Webhooks URLを貼り付けてください。「Content type」は「application/json」を指定します。
8. 「Add webhook」をクリックします。

以上でWebUIアプリのWebhook設定は完了です。
WebAPIアプリも、Topologyで「worklog-api」を選択する以外はすべて同じ作業でWebhook設定が可能ですので、設定してください。

## 2. ソースコードの修正とgit push

Webhookの設定が完了したら、実際にソースコードを修正しgithubにpushしてみましょう。

1. [web/worklog/public/index.html](web/worklog/public/index.html)をエディタで開きます。
2. `<title>`タグの文字列を、「作業ログ」に修正し、保存します。
3. github にpushします。適宜完了にあわせて修正してください。
    * git add web/worklog/public/index.html
    * git commit -m "change index title"
    * git push

ここまで完了したら、すぐにOpenShiftWebコンソール画面を開いてください。

## 3. ビルドの自動実行と更新

githubへのpushが完了したら、すぐにOpenShiftWebコンソール画面を開きます。

ビルドが始まっていることを確認しましょう。

1. OpenShift Webコンソール画面にて、UIをDeveloperモードにし、Topologyを選択します。
2. 「worklog-ui」を選択し、ResourcesタブのBuilds下にある「BC worklog-ui」を確認します。
3. 新しいビルドが始まっていることを確認します。

ここで、もしビルドが始まっているのであれば設定は成功です。

ビルドが終わったら、アプリケーションのデプロイも完了しています。アプリの動作確認を行いましょう。

以上で、ソースコードをgithubにpushするだけで、テストやリリース工程で利用できるアプリケーションがビルドできること、アプリケーションがデプロイできることを確認することができました。

# その他参考リンク

* [Flask](http://flask.pocoo.org/)
* [Swagger Editor](https://editor.swagger.io/)

# 参考情報

* **Container Code Patterns**:コードパターンを楽しんでますか? 他の[Container Code Patterns](https://developer.ibm.com/patterns/category/containers/)もチェックしてみてください。
* **Python Code Patterns**:コードパターンを楽しんでますか? 他の[Python Code Patterns](https://developer.ibm.com/patterns/category/python/)もチェックしてみてください。

# License

This code pattern is licensed under the Apache Software License, Version 2.  Separate third party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](http://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](http://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
