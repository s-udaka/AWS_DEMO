# AWSシステム構築手順

## 1. ネットワーク構築

### VPC作成
1. マネジメントコンソールから**VPC作成**をクリック
2. 以下のように設定

|項目|設定値|説明|
|---|---|---|
|Name tag|udaVPC|このVPCの名前。自分が管理するための任意の名前を付ける。|
|IPv4 CIDR block|10.0.0.0/16|実際は、必要なネットワークの大きさを考慮して決定する。|
|IPv6 CIDR block|No IPv6 CIDR Block|IPv6のCIDRブロックを使用するかどうか。|
|Tenancy|Default|このVPCが作られるサーバ（ハードウェア）をこのVPCだけで専有するかどうか。|

### インターネットゲートウェイ作成
1. [VPC]のページのサイドメニューの[インターネットゲートウェイ]をクリックし、上部にある[インターネットゲートウェイの作成]ボタンをクリック。
2. 以下のように設定

|項目|設定値|説明|
|---|---|---|
|Name tag|udaIGW|このインターネットゲートウェイの名前。自分が管理するための任意の名前を付ける。|

### インターネットゲートウェイをVPCにアタッチ
1. インターネットゲートウェイの一覧で、先ほど作成したインターネットゲートウェイをチェックし、ページ上部の[アクション]→[VPCにアタッチ]をクリック。
2. 先ほど作成した[udaVPC]のVPCを選択してアタッチする。

### サブネット作成
1. [VPC]のページのサイドメニューの[サブネット]をクリックし、上部にある[サブネットの作成]ボタンをクリック。
2. 以下のように設定

|項目|設定値|説明|
|---|---|---|
|Name tag|uda_subnet_public|このサブネットの名前。自分が管理するための任意の名前を付ける。|
|VPC|先ほど作成したudaVPC|このサブネットが作られるVPC。|
|アベイラビリティーゾーン|(お好みで)|このサブネットが作られるアベイラビリティーゾーン|
|IPv4 CIDR ブロック|10.0.0.0/24|実際は、必要なネットワークの大きさを考慮して決定する。|

### ルートテーブル作成
#### インターネットゲートウェイにルーティングする設定のルートテーブルを作成し、それをサブネットに関連付けることでサブネット内のサーバはインターネットに接続できるようになります。
1. ルートテーブルの作成
   - [VPC]のページの
サイドメニューの[ルートテーブル]をクリックし、上部にある[Create route table]ボタンをクリック。
   - 以下のように設定

|項目|設定値|説明|
|---|---|---|
|Name tag|uda_route_table_public|このルートテーブルの名前。自分が管理するための任意の名前を付ける。|
|VPC|先ほど作成したudaVPC|このルートテーブルが作られるVPC。|

2. ルーティングの変更
   - 先ほど作成したルートテーブルはデフォルトのルーティング設定になっているため、
ルーティングを追加します。
   - ルートテーブルの一覧で先ほど作成したルートテーブルをチェックし、ページ上部の[アクション]→[Edit routes]をクリック。
   - [Destination] = [0.0.0.0/0]、[Target] = {先ほど作成した[udaIGW]のインターネットゲートウェイ}のルーティングを追加する。

### ルートテーブルをサブネットに関連付け
1. サブネットの一覧で先ほど作成した[uda_subnet_public]をチェックし、ページ上部の[アクション]→[ルートテーブルの変更]をクリック。
2. [ルートテーブルID]に先ほど作成した[uda_route_table_public]のルートテーブルを設定する。

**これで、このサブネット内のサーバはインターネットに接続できる状態になりました。**

## 2. webサーバ構築

### EC2インスタンス作成
1. マネジメントコンソールの[EC2]のページのサイドメニューの[インスタンス]をクリックし、上部にある[インスタンスの作成]ボタンをクリック。
2. サーバのOSにAmazon Linux 2 AMIを選択します。
3. インスタンスのスペックt2.microを選択します。
4. インスタンスの詳細設定を下記のように設定します。下記の3項目だけ間違えないように設定し、それ以外はデフォルト設定で問題ないです。

|項目|設定値|説明|
|---|---|---|
|ネットワーク|{先ほど作成した[udaVPC]]のVPC}|このインスタンスが作られるVPC。|
|サブネット|{先ほど作成した[uda_subnet_public]]のサブネット}|このインスタンスが作られるサブネット。|
|自動割り当てパブリック IP|有効|	パブリックIPを利用するかどうか。|

5. インスタンスのストレージを設定します。実際は、サーバの用途に合わせてサイズやボリュームタイプを検討しますが、今回はデフォルト設定のままで問題ないです。
6. インスタンスを管理するために、任意のタグ、値を設定します。今回はNameタグ[uda_webserver]だけ追加します。
7. [セキュリティグループの割り当て]で[新しいセキュリティグループを作成する]を選択し、下記のように設定します。

|項目|設定値|説明|
|---|---|---|
|セキュリティグループ名|uda_sg_web|このセキュリティグループの名前|
|説明|uda_sg_web|このセキュリティグループの説明|
|SSH|マイIP|自分の場所のIPからSSHでの通信を許可します。|
|HTTP|マイIP|自分の場所のIPからHTTPでの通信を許可します。|

8. これまでの設定内容を確認し、[起動]ボタンをクリックします。
9. インスタンスにSSH接続するために必要なキーペアの設定画面が表示されるので、既存のキーペアを選択する場合は、[既存のキーペア]を選択し、新しくキーペアを作成する場合は、[新しいキーペアの作成]をクリックし、[キーペア名]に任意の名前を入力し、[キーペアのダウンロード]をクリックします。
10. 最後に[インスタンスの作成]ボタンをクリックしたら、EC2インスタンスが作成されます。
11. インスタンスの一覧で、作成したインスタンスの[インスタンスの状態]が[running]になっていればEC2インスタンスが正常に起動しています。

### EC2インスタンスにnginxをインストール
1. 先ほど作成したEC2インスタンスにnginxをインストールするため、まずはインスタンスにSSH接続します。
2. Tera Termなどのターミナルソフトを利用し、下記の3つを設定してSSH接続します。

|項目|設定値|説明|
|---|---|---|
|接続先ホスト|{先ほど作成した[uda_webserver]のEC2インスタンスのパブリックIP}|EC2のページでさっき作ったインスタンスのパブリックIPを確認する。|
|ログインユーザ名|ec2-user|Amazon Linux 2 AMIで作成したインスタンスはデフォルトで[ec2-user]というユーザが作成される。|
|SSH認証鍵|{先ほど作成したor既存のキーペアの～.pemの鍵ファイル}|EC2作成最後のステップでダウンロードしたor既存の鍵ファイル。|

3. SSH接続できたら、下記のコマンドを実行してnginxをインストールします。
```
sudo amazon-linux-extras install nginx1
```
4. 下記のコマンドを実行してnginxを起動、次回以降はサーバー起動時に自動でnginxが起動するように設定します。
```
sudo systemctl start nginx
sudo systemctl status nginx
sudo systemctl enable nginx
systemctl is-enabled nginx
```
5. ブラウザのアドレスバーにEC2インスタンスのパブリックIPを入力し、nginxのデフォルト画面が表示されればwebサーバの構築完了です。

## 3.webサーバにdjango環境構築
#### 上記で作成したwebサーバにSSHログインし、下記手順にそってセットアップしていきます。
1. パッケージインデックスを更新し、Python 3 がホストにすでにインストールされているかどうかを確認します。
```
yum check-update
yum list installed | grep -i python3
```
2. Amazon Linux 2 に Python 3 をインストールする（Python 3 がまだインストールされていない場合）
```
sudo yum install python3 -y
```
3. ec2-user ホームディレクトリの下に仮想環境を作成する
```
python3 -m venv myenv
```
4. Python3の仮想環境を有効化
```
source myenv/bin/activate
```
5. 仮想環境内に最新の pip モジュールがインストールされていることを確認してから、必要なモジュールをインストール
```
pip install pip --upgrade
pip install django gunicorn
```
6. 仮想環境から一旦抜ける
```
deactivate
```
7. ログイン時に仮想環境を自動的にアクティブ化するため、~/.bashrc ファイルに追記します。
```
echo "source ${HOME}/myenv/bin/activate" >> ${HOME}/.bashrc
```
8. ホームディレクトリの ~/.bashrc ファイルをソースして、環境の bash 環境をリロードします。リロードすると、仮想環境が自動的にアクティブになります。
```
source ~/.bashrc
```
9. テストアプリをgitからcloneしてくる
```
sudo yum update -y
sudo yum install git -y
git clone https://github.com/~
```
10. テストアプリの最新版を取得する
```
git pull
git checkout -b develop origin/develop
git branch
```
11. 仮想環境から抜ける
```
deactivate
```
## 4.Gunicornの設定
#### GunicornとはRubyで利用されるAPサーバのUnicornをもとにして作られたPython向けのAPサーバです。
1. Gunicornのserviceファイルを編集(作成)します。
```
sudo vim /etc/systemd/system/gunicorn.service
```
2. おそらくファイルが存在していないので、以下を貼り付けてください。
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/home/ec2-user/aws_django_test/test_django
ExecStart=/home/ec2-user/myenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/ec2-user/aws_django_test/test_django.sock test_django.wsgi:application

[Install]
WantedBy=multi-user.target
```
3. Gunicorn serviceの自動起動を設定します。
```
sudo systemctl start gunicorn.service
sudo systemctl enable gunicorn
```
## 5.Nginxの設定
1. Nginxの設定ファイルを作成します
```
sudo vim /etc/nginx/conf.d/aws_django_test.conf
```
2. 設定ファイルの内容は以下の通り（server_nameはEC2のパブリックIP）
```
server {
        listen 80;
        server_name EC2のパブリックIP;

        location = /favicon.ico {access_log off; log_not_found off;}
        location /static/ {
                root /home/ec2-user/aws_django_test;
        }

        location / {
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_pass http://unix:/home/ec2-user/aws_django_test/test_django.sock;
        }
}
```
3. nginx.confの内容を一部変更する（実行ユーザーを変更する）
```
# user nginx;
user ec2-user;

```
4. Nginxをリスタートして設定を反映させる
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
5. 最後にGunirornをリスタートしておきましょう。
```
sudo systemctl restart gunicorn
```
6. ブラウザからEC2インスタンスのパブリックIPにアクセスして、テストアプリが表示されればOK
