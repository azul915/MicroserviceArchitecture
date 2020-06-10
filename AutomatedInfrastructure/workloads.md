# リソースの種類
* Pod
* (ReplicationController <- 廃止予定)
* ReplicaSet
* Deployent
* DaemonSet
* Job
* CronJob

Podを最小単位として、それらを管理する上位リソースがある親子関係
* Pod(Tier1) <- 管理 - ReplicaSet(Tier2) <- 管理 - Deployment(Tier3)
* Pod(Tier1) <- 管理 - Job(Tier2) <- 管理 - CronJob(Tier3)

# Pod
* Workloadsリソースの最小単位
* 1つ以上のコンテナから構成
* 同じPodに含まれるコンテナ同士はネットワーク的に隔離されておらず、IPアドレスを共有している（= Pod内のコンテナはお互いいlocalhost宛で通信することが可能）
* 多くの場合は1つのPodに一つのコンテナを内包する
* メインのコンテナ以外に依存関係の都合等で、複数のコンテナを同一Podで管理することもある
* サブコンテナ一例（プロキシの役割、設定値の動的な書き換えの役割、ローカルキャッシュ用コンテナ、SSL終端用コンテナ）-> サイドカー、アンバサダー、アダプタといったデザインパターン

# ReplicaSet/ReplicationController
* ReplicaSet/ReplicationControllerはPodのレプリカを作成して、指定した数のPodを維持し続けるリソース
* ReplicationController -> ReplicaSetにリネームされて一部機能が追加
* ノードやPodに障害が発生した場合でも、Pod数が指定された数を満たすように別のノードでコンテナを起動する
* spec.selectorに設定した値を、spec.template.metadata.labelsで探して、都度レプリカを追加する
* ReplicaSetをスケーリングするには
   * マニフェストを書き換えて、`kubectl apply -f`（Infrastructure as Codeを守る意味で基本的にマニフェストの書き換えで行う）
   * `kubectl scale`でスケーリング処理を行う（Deployment, StatefulSet, Job, CronJob）でも可能

# Deployment
* 複数のReplicaSetを管理することで、ローリングアップデータやロールバックなどを実現するリソース
* DeploymentがReplicaSetを管理して、ReplicaSetがPodを管理する3層の親子関係
* Deploymentを使うと、新しいReplicaSet上でコンテナが起動したことやヘルスチェックが通っていることを確認しながら切り替えを行ってくれる
* Podでデプロイした場合にはPodに障害が発生した際に自動でPodが再作成されることはない
* ReplicaSetでデプロイした場合にはローリングアップデートなどの機能の利用ができない

# Job
* コンテナを利用して一度限りの処理は実行させるリソース
* N並行で実行しながら、指定した回数のコンテナの実行（正常終了）を保証するリソース
* ReplicaSetとの違いとして、「起動するPodが停止することを前提にして作られているかどうか」がある。
* `kubectl get jobs`で正常に終了したPodの数（SUCCESSFUL）を表示する（ReplicaSetではREADYなコンテナ数を表示していたが）
* spec.template.spec.restartPolicyにOnFaiureまたはNeverを指定する必要がある
   * OnFailure: 再度同一のPodを利用してJobを再開する
      * Podが起動するノードやIPアドレスが変わることはないが、PersistentVolume（永続化領域）やKubernetesNode上の領域（hostPath）をマウントしていない場合にはデータ自体は紛失する
   * Never: Pod障害時に新規のPodが作成される
* completions: 成功回数の指定（defaultで1）
* parallelism: 並列度の指定（defaultで1）

# CronJob