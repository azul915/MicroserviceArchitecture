# DevOpsってなに
## 概要
* DevOpsとは「開発チーム（Development）と運用チーム（Operations）がお互いに協調し合うことで、開発・運用するソフトウェア／システムによってビジネスの価値をより高めるだけでなく、そのビジネスの価値をより確実かつ迅速にエンドユーザーに届け続ける」という概念。（出典[「DevOpsとは何か？ そのツールと組織文化、アジャイルとの違い」](https://www.buildinsider.net/enterprise/devops/01)）
* Devの役割が「システムに新しい機能を追加する」であるのに対し、Opsの役割は「システムの安定稼働」のような対立構造が作られている場合、「ソフトウェアやシステムによってビジネスの価値を高めたり、ビジネスの価値をより確実かつ迅速にエンドユーザーに届け続ける」というようなミッションの達成の妨げになってしまうようなケースを背景としている（このケースと以下の取り組みはとあるイベントで提唱されたもの）
* こういった問題を解決するために、ツールと開発文化の観点でDevとOps間で以下のようなことが取り組まれる様になっている
* CODE -> BUILD -> TEST -> RELEASE -> DEPLOY -> OPERATE -> MONITOR -> PLAN -> CODE -> ...
* もともとはアジャイル開発のムーブメントから生まれた
## ツール
* インフラの自動化/Infrastructure as Code
   * コードによってインフラの構築を自動化する
   * Ansible, Chef, Puppet, Docker、Kubernetes
* ソースコードをバージョン管理
   * 同じバージョン管理システムをDevとOps間で共有する
   * Git
* 自動デプロイ
   * 手動デプロイではなく、ビルドやデプロイを宣言的に定義する
   * Jenkins, CircleCI, GitLabCI/CD
* メトリクスの共有
   * 取得したメトリクスの結果をダッシュボードでDevとOps間で共有する
   * DataDog、Prometheus, Zabbix, CloudWatch
* IRCやチャットツールのBot
   * 自動ビルドやデプロイのログや通知を自動化して情報を相互共有する
   * Slack, Teams

## カルチャー
* 相互尊重
* 相互信頼
* 失敗に対して健全な態度を摂る
* 非難禁止

# デプロイ方法
カナリアリリース
新バージョンのリリース時に旧バージョンを並行的に稼働させながら、一部のユーザーやリクエストのみ新バージョンにアクセスさせるデプロイ方法のこと
例えば、ロードバランサーによってリクエストごとに、新バージョンと旧バージョンアプリへのアクセスを振り分ける方法がある
問題がないことを確認しながら段階的に新バージョンへのアクセス振り分けを増やし、全てのリクエストが新バージョンへ振り分けられるタイミングで旧バージョンアプリケーションを捨てる