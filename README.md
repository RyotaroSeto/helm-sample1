# helm-sample1

- helm install full-coral ./mychart
- helm uninstall full-coral
- helm get manifest full-coral
- helm install geared-marsupi ./mychart --dry-run --debug
- subchart
  - cd mychart/charts
  - helm create mysubchart
  - rm -rf mysubchart/templates/\*
  - helm install --generate-name --dry-run --debug mychart/charts/mysubchart

## helm についての基本

- 「nginx がほしい」と願えば上記の YAML を生成してくれる
- YAML の生成は、Chart（チャート）と Release（リリース）で実現
  - Chart は、Kubernetes でよく作られる Pod や Service などの YAML ファイルのひな型を固めて圧縮したもの
  - Release は、Chart に基づいてコピー生成された Pod や Service などの YAML ファイル

nginx のチャートは大抵すでに存在するため、`helm install`すれば新しいリリースを作成してくれて、ついでに Kubernetes にデプロイしてくれる。
生成した YAML ファイル自体に目を通すこともなく、Kubernetes 上に nginx が構築される

## 使い方

### 1. 作成したマニフェストを helm 化

`helm install リリース名 対象ディレクトリ`

### 2. helm 化した全てのマニフェストのを取得

`helm get manifest リリース名`

### 3. リリースしたものを削除

`helm uninstall リリース名`

### 4. name リソースのコーディングについて

name:をリソースにハードコーディングすることは、通常、悪い習慣であると考えられています。名前はリリースごとに一意である必要がある

- `{{ .Release.Name }}-configmap`を使う
  - `{{ .Release.Name }}`はリリース名が入る

## 組み込みオブジェクト

### Release

- Release.Name：リリース名
- Release.Namespace: リリースされる名前空間 (マニフェストがオーバーライドしない場合)
- Release.IsUpgradetrue:現在の操作がアップグレードまたはロールバックの場合に設定されます
- Release.IsInstalltrue:現在の操作がインストールの場合、これは に設定されます。
- Release.Revision: このリリースのリビジョン番号。インストール時は 1 で、アップグレードとロールバックのたびに増加します。
- Release.Service: 現在のテンプレートをレンダリングしているサービス

### Values

- values.yaml:ファイルおよびユーザー指定のファイルからテンプレートに渡される値。デフォルトでは Values 空です。

### Chart

- ファイルの内容 Chart.yaml。内のすべてのデータ Chart.yaml にここからアクセスできます。たとえば{{ .Chart.Name }}-{{ .Chart.Version }}、 を出力します mychart-0.1.0

### Files

これにより、チャート内のすべての非特殊ファイルへのアクセスが提供されます

### Capabilities

Kubernetes クラスターがサポートする機能に関する情報を提供

### Template

実行中の現在のテンプレートに関する情報

### lookup 機能の使用

| kubectl                              | ルックアップ関数                         |
| ------------------------------------ | ---------------------------------------- |
| kubectl get pod mypod -n mynamespace | lookup "v1" "Pod" "mynamespace" "mypod"  |
| kubectl get pods -n mynamespace      | lookup "v1" "Pod" "mynamespace" ""       |
| kubectl get pods --all-namespaces    | lookup "v1" "Pod" "" ""                  |
| kubectl get namespace mynamespace    | lookup "v1" "Namespace" "" "mynamespace" |
| kubectl get namespaces               | lookup "v1" "Namespace" "" ""            |

### Notes.txt ファイルの作成

- インストールに関するメモをチャートに追加するには、templates/NOTES.txt ファイルを作成するだけ

### サブチャート

- サブチャートは親の値にアクセスできない
- 親チャートはサブチャートの値をオーバーライドできる
- Helm は全てのチャートからアクセスできるグローバル値の概念がある

### .helmignore

- 「\*\*」構文はサポートされていません。
- 末尾のスペースは常に無視されます
- 「!」はサポートされていません。
- デフォルトではそれ自体を除外しません。明示的なエントリを追加する必要があります

### デバッグテンプレート

- helm lint グラフがベスト プラクティスに従っていることを確認するための頼りになるツール
- helm template --debug チャート テンプレートのレンダリングをローカルでテスト
- helm install --dry-run --debug サーバーにテンプレートをレンダリングさせ、結果のマニフェスト ファイルを返す
- helm get manifest: サーバーにどのようなテンプレートがインストールされているかを確認する良い方法

### チャート管理

```
helm create <name>                      # チャートで使用される一般的なファイルやディレクトリとともにチャート・ディレクトリを作成
helm package <chart-path>               # チャートをバージョン管理されたチャート・アーカイブ・ファイルにパッケージする
helm lint <chart>                       # チャートを検査し、可能性のある問題を特定するためにテストを実行
helm show all <chart>                   # チャートを点検し、その内容を列挙
helm show values <chart>                # values.yamlファイルの内容を表示
helm pull <chart>                       # ダウンロード／プルチャート
helm pull <chart> --untar=true          # trueに設定すると、チャートをダウンロードした後にuntarする。
helm pull <chart> --verify              # 使用前にパッケージを確認する
helm pull <chart> --version <number>    # Default-latestを使用する場合、使用するチャート・バージョンの制約を指定する。
helm dependency list <chart>            # チャートの依存関係のリストを表示する
```

### アプリのインストールとアンインストール

```
helm install <name> <chart>                           # 名前を指定してチャートをインストールする。
helm install <name> <chart> --namespace <namespace>   # 特定の名前空間にチャートをインストールする。
helm install <name> <chart> --set key1=val1,key2=val2 # コマンドラインで値を設定 (カンマで区切って複数指定可能)
helm install <name> <chart> --values <yaml-file/url>  # 指定した値でチャートをインストールする。
helm install <name> <chart> --dry-run --debug         # チャートを検証するためにテストインストールを実行する (p)
helm install <name> <chart> --verify                  # パッケージを使う前に検証する
helm install <name> <chart> --dependency-update       # チャートをインストールする前に依存関係がなければ更新する。
helm uninstall <name>                                 # リリースをアンインストールする
```

### その他コマンド

https://helm.sh/docs/intro/cheatsheet/
