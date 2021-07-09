# OpenShiftへの移行作業ログ

* requirement.txtの指定バージョンが古いようで、自環境にはインストールできない
  * 指定バージョン無視で最新バージョンを入れる(=>動いた)

      ```sh
      pip install pyYAML simple-crypt pymongo Flask-Cors Flask flasgger docker decorator
      ```

* oc loginのためにOpenShiftコンソールの右上「Copy Login Command」からoc loginのコマンドコピー

    ```sh
    oc login --token=sha256~xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --server=https://c100-e.jp-tok.containers.cloud.ibm.com:31616
    ```

* oc new-project worklog
* DBを作成する
  * oc apply -f deploy-mongodb.yml

* まずはWebAPIアプリをデプロイする
  * OpenShift - Developer - +Add - from Git でデプロイ
    * <https://github.com/tty-kwn/worklog.git>
    * /
    * Routes: 5000
  * 以下のエラーでpod error。

      ```sh
      Please set either APP_MODULE, APP_FILE or APP_SCRIPT environment variables, or create a file 'app.py' to launch your application.
      ```

  * <https://github.com/sclorg/s2i-python-container/tree/master/3.8>
  * APP_MODULEはgunicornの`wcgi.py`などを指定するらしい
  * APP_FILEはアプリのエントリポイントファイルを指定する(デフォルトは`app.py`)
    * `app/__init__.py`を指定すればいいかもしれない
  * APP_SCRIPTはデフォルトは`app.sh`
    * `app.sh`を作成し、元のDockerfileにある`python -m app.__init__`を記述

        ```sh
        /usr/libexec/s2i/run: line 40: /opt/app-root/src/app.sh: Permission denied
        ```

    * [stackoverflow](https://stackoverflow.com/questions/46091601/cannot-chmod-file-on-openshift-online-v3-operation-not-permitted)を参考に、以下のコマンドを実行

        ```sh
        chmod +x app.sh 
        git update-index --chmod=+x app.sh 
        git add app.sh 
        git commit 
        git push
        ```

    * s2i buildしなおして成功！

* 続けてWebUIアプリをデプロイする
* OpenShift - Developer - +Add - from Git でデプロイ
  * <https://github.com/tty-kwn/worklog.git>
    * /web/worklog
    * Routes: 3000
    * Deployment - Env: 'REACT_APP_APP_SERVER'
      * <http://worklog3-worklog.kawano-cluster-tok04-4216c78965aaa521311d0371fde68bf9-0000.jp-tok.containers.appdomain.cloud>
  * -> OK

* アプリ動作確認開始
* Create Account画面にてエラー
  * <http://worklog3-worklog.kawano-cluster-tok04-4216c78965aaa521311d0371fde68bf9-0000.jp-tok.containers.appdomain.cloud/api/v1/user/create> HTTP 500

    ```sh
    [2021-07-06 04:43:04,858] ERROR in app: Exception on /api/v1/user/create [PUT]
    Traceback (most recent call last): File "/opt/app
    …
    File "/opt/app-root/lib64/python3.8/site-packages/Crypto/Random/_UserFriendlyRNG.py", line 77, in collect t = time.clock() 
    AttributeError: module 'time' has no attribute 'clock' 
    172.30.149.145 - - [06/Jul/2021 04:43:04] "[35m[1mPUT /api/v1/user/create HTTP/1.1[0m" 500 -
    ```

  * <https://stackoverflow.com/questions/58569361/attributeerror-module-time-has-no-attribute-clock-in-python-3-8>
    * "PyCrypto is dead as mentioned on project issue page"
  * Pycryptoを消してPycryptodomeに置き換え

    ```sh
    pip3 uninstall PyCrypto && pip3 install -U PyCryptodome
    pip freeze |grep pycrypto >> requirements.txt
    ```

  * 次のエラー

    ```sh
    [2021-07-06 05:10:12,347] ERROR in app: Exception on /api/v1/user/create [PUT] 
    Traceback (most recent call last): File "/opt/app
    …
    File "/opt/rh/rh-python38/root/usr/lib64/python3.8/ctypes/__init__.py", line 391, in __getitem__ func = self._FuncPtr((name_or_ordinal, self)) 
    AttributeError: /opt/app-root/lib64/python3.8/site-packages/Crypto/Util/../Hash/_SHA256.cpython-38-x86_64-linux-gnu.so: undefined symbol: SHA256_init 
    172.30.149.145 - - [06/Jul/2021 05:10:12] "[35m[1mPUT /api/v1/user/create HTTP/1.1[0m" 500 -
    ```

  * どうもならないのでソースコードを修正
  * 参考：<https://qiita.com/penta2019/items/a500630608960752a914>
    * simple-cryptを外しpycryptodomeを直接利用するように修正
    * ←SHA256のライブラリが足りないので、利用しないようにする
  * -> OK
