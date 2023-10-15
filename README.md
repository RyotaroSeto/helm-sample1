# helm-sample1

- helm install full-coral ./mychart
- helm uninstall full-coral
- helm get manifest full-coral
- helm install geared-marsupi ./mychart --dry-run --debug

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
