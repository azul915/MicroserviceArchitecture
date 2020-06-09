# リソース
* クラスタを操作するために`kubectl`と`マニフェストファイル`を用いて、マスターノードに「リソース」の登録を行う（kubectlはマニフェストファイルの情報を元にマスターノードが持つAPIServerにリクエストを行い、Kubernetesの操作を行う。

* 「リソース」を登録することで、コンテナの実行やロードバランサの作成が非同期におこなわれる

| 種別 | 概要 | 例
| --- | --- | --- |
| Workloads リソース | コンテナの実行に関するリソース | Pod, ReplicationController, Deployment, Cronjob 等 |
| discovery & LB リソース | コンテナを外部公開するようなエンドポイントを提供するリソース。。コンテナのサービスディスカバリやクラスタの外部からもアクセス可能なエンドポイントなどを提供する。 | Service, Ingress |
| Config & Strage リソース | 設定/機密情報/永続化ボリュームなどに関するリソース。設定や機密データをコンテナに埋め込んだり、永続化ボリュームを提供する。 | Secret, ConfigMap, PersistentVolumeClaim |
| Cluster  リソース | セキュリティやクラスタなどに関するリソース。セキュリティ周りの設定やポリシー、クラスタの管理性を高める機能のためのリソースなど | Node, Namespace, Role, ClusterRoleなど |
| Metadata リソース | クラスタ内の他のリソースを操作するためのリソース。クラスタ内の他のリソースの動作を制御するためのリソース。 | LimitRange, HorizontalPodAutoscalerなど
# kubectl
* clusters: 接続先クラスタ情報
* users: 認証情報
* context: clusterとuserのペアとnamespaceを指定したものの定義 -> Contextを切り替えていくことで、複数の環境を複数の権限で操作できる

# 操作
## Podの一覧を取得
```bash
$ kubectl get pods
```
## マニフェストを指定して存在するリソースを削除
```bash
$ kubectl delete -f manifest.yaml
```

## マニフェストの変更点をリソースに適用
* 変更差分がある場合: 変更点の反映処理
* リソースが存在しない場合: リソースの作成（リソースの作成については、`kubectl create`コマンドがあるが、基本的には作成時も`kubectl apply`コマンドを使える）
```bash
$ kubectl apply -f manifest.yaml
```

## マニフェスト適用時に、削除されたリソースを検知して、自動的に削除する
* ディレクトリ配下に複数のマニフェストフィアルを使用している場合、1つのマニフェストファイルに複数リソースを定義している場合の両方ともに使える
* ラベルに一致する全リソースのリスト（ラベルが指定されていない場合は全てのリソース）から「kubectl apply」で渡したマニフェストにないものを全て削除する。
```bash
$ kubectl apply --prune
```

## コンテナイメージを確認する
```bash
$ kubectl describe nodes my-node
$ kubectl describe pod(s) my-pod
```

## 実際のリソースの使用量の確認
* `kubectl describe`コマンドで確認できるリソースの使用量は、k8sがPodに確保した値なので、実際にPod内のコンテナが使用しているリソースの使用量は`kubectl top`で確認する
* ノードのリソース使用量は`kubectl top node`を使用する
```bash
$ kubectl top node
```
## Pod上でのコマンドの実行
```bash
# Pod内ののコンテナで/bin/shを実行（終了時exit）
$ kubectl exec -it sample-pod /bin/sh

# 複数コンテナがはいいいたPodの特定のコンテナで/bin/shを実行
$ kubectl exec -it sample-pod -c nginx-container /bin/sh
```

## Podのログ確認
```bash
# Pod内のコンテナのログを出力
$ kubectl logs sample-pod

# 複数コンテナが入った特定のコンテナのログを出力
$ kubectl logs sample-pod -c nginx-container
```

## kubectlのログレベル指定
* kubectlコマンド実行時に`-v`オプションでログレベルを指定できる
```bash
$ kubectl -v=6 get pod
```

# マニフェストファイルについて
* 一つのマニフェストファイルの中に複数のリソースを記述することもできる。
   * あるサービスに関する様々な種類のリソースを1つのファイルにまとめる使い方
   * 実行順序を厳密にしたい場合、や、リソース間の結合度が強い場合はリソースファイルをまとめるといい
   * マニフェストに複数のリソースを記述する場合は「---」で区切る
* 複数のマニフェストファイルを同時に適用できる。
   * ディレクトリ配下に適用したい複数のマニフェストファイルを配置しておき、`kubectl`コマンド実行時にディレクトリを指定する
   * ファイル名順にマニフェストファイルが適用されるため、順序制御を行いたいときは、先頭にインデックス番号を書くと良い。

# ラベル
* metadata.labelsに設定することができるメタデータ
* リソースを分別するための情報
* 数多くのリソースに対して、同一ラベルでグルーピングして処理したり、何からの処理に対する条件分岐に利用する
