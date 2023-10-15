# helm-sample1

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
