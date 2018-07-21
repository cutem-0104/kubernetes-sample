# brewとminikubeのインストール

## 事前準備

- virtualboxをインストールしておく

```bash
# minikubeのインストール
brew cask install minikube
# minikubeとkubectlのバージョン確認
minikube version
kubectl version --client
# minikubeが使うipの確認
minikube ip
```

## minikubeのVM起動

```bash
# minikubeの起動
minikube start
# コマンドが正常動作するか確認
kubectl version
```

## minikubeのQuickStart

```bash
# 公式サンプルイメージでDeployment作成
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
# Service作成
kubectl expose deployment hello-minikube --type=NodePort
# Deploymentに紐づくPodの状態を客員
kubectl get pod # RunningのStatusになっていたら起動完了
# エンドポイントにアクセス
curl $(minikube service hello-minikube --url)
# urlを調べる
minikube service hello-minikube --url
# 表示されたurlにアクセスし、curlコマンドの結果と同じ内容が表示されることを確認する
```

## Kubernetesの管理画面

```bash
# 下記コマンドを使うことでkubectlコマンドで接続設定されているリモートのKubernetesとローカルのポートをプロキシ接続したサービスを起動できる
kubectl proxy
```

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
にアクセスすると管理画面が表示される

## キャッシュサーバーを立てる

```bash
# ファイルをもとにDeploymentとServiceを作成
# 純粋に作成のみの場合はkubectl create
# 作成または変更の適用を行いたい場合はkubectl apply
# ymlファイルを指定して環境を構築する場合は-fオプションを付ける
kubectl apply -f cache.yml
# 管理画面へアクセスする場合はサーバーへのプロキシを張る
kubectl proxy &
```

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy
にアクセスし、緑色のチェックマークとともにDeployment,Pod,Serviceが表示されていれば無事に動作している

```bash
# キャッシュサーバーへの接続確認
minikube service todo-cache-service --url
# redisをインストール
brew install redis
# redis-cliで接続確認
# 実際は最初のコマンドで表示されたipとポートを指定する
redis-cli -h 192.168.99.100 -p 30431
```

## データベースサーバーを立てる

```bash
# コンテナの起動
kubectl apply -f db.yml
```

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy
にアクセスし、緑色のチェックマークとともにDeployment,Pod,Service Secretが表示されていれば無事に動作している

```bash
# mysqlのipとポートを確認
minikube service todo-db-service --url
# mysqlのインストール
brew install mysql
# mysqlの接続
mysql -h 12.168.99.100 -P 32065 -usampleuser -psamplePass
# テーブルの作成
USE sampleDb;
CREATE TABLE `todos` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(128) DEFAULT NULL,
  `image_url` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
SHOW tables;
```

## ファイルサーバーを立てる

```bash
# コンテナの起動
kubectl apply -f file.yml
```

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy
にアクセスし、緑色のチェックマークとともにDeployment,Pod,Service PersistentVolumeClaim,Secretが表示されていれば無事に動作している

```bash
# minioサービスの接続確認
minikube service todo-s3-service --url
```

表示されたurlにアクセスしminioの画面が表示されることを確認
アクセスできたらkubernetes-sample-bucketという名前のバケットを作成する

```bash
# ローカル環境にminioサーバーを設定するためのmcクライアントをdockerで起動する
docker run --rm -it --entrypoint /bin/sh minio/mc:RELEASE.2017-06-15T03-38-43Z
# コンテナ内で実行
mc ls
vi /root/.mc/config.json # localブロック内を編集 urlはminioのurlを入れる
mc ls local
mc policy public local/kubernetes-sample-bucket
```

## アプリケーションサーバーをたてる

```bash
# アドオンの状態を確認する
minikube addons list
# ingressの状態がdesabledになっている場合は有効にする
minikube addons enable ingress
# コンテナの起動
kubectl apply -f application.yml
```

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy
にアクセスし、緑色のチェックマークとともにDeployment,Pod,Service Ingressが表示されていれば無事に動作している

```bash
minikube service todo-application-service --url
```

表示されたURLにアクセスして表示されること、TODOが作成できることを確認