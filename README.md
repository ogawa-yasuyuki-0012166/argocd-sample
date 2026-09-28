# Argo CD + CircleCI sample

GitHub への `main` ブランチの push を CircleCI が検知し、コンテナイメージを GitHub Container Registry (GHCR) へ push します。続けて CircleCI が `k8s/base/kustomization.yaml` のイメージタグを更新し、Argo CD がその Git の変更を kind クラスタへ自動同期します。

## 事前準備

1. GitHub に `argocd-sample` リポジトリを作成し、このリポジトリを `main` ブランチとして push します。
2. `argocd/application.yaml` の `REPLACE_WITH_GITHUB_OWNER` を GitHub ユーザー名または Organization 名に変更します。
3. `k8s/base` 内のイメージ名は、最初の CircleCI 実行時に `GHCR_OWNER` の値で自動置換されます。
4. CircleCI で GitHub リポジトリをプロジェクトとして登録し、Project Environment Variables に次を設定します。

| 変数 | 値 |
| --- | --- |
| `GHCR_OWNER` | GitHub ユーザー名または Organization 名 |
| `GHCR_TOKEN` | `write:packages` 権限を持つ GitHub fine-grained PAT |
| `GITOPS_PUSH_TOKEN` | このリポジトリへの `Contents: Read and write` 権限を持つ GitHub fine-grained PAT |

GHCR パッケージを private にする場合は、kind クラスタが pull できる `imagePullSecret` も Deployment に設定してください。最初の検証では GHCR パッケージを public にするのが簡単です。

## Argo CD への登録

プレースホルダーを置換して GitHub へ push した後、WSL で実行します。

```bash
kubectl apply -f argocd/application.yaml
kubectl -n argocd get applications.argoproj.io argocd-sample
kubectl get deployment,service argocd-sample
kubectl port-forward service/argocd-sample 8080:80
```

ブラウザで `http://localhost:8080` を開くとサンプルページを確認できます。

## ローカルコンテナ確認

WSL の Docker が Docker Hub へ接続できる状態で、次を実行します。

```bash
docker build -t argocd-sample:local .
docker run --rm -p 8080:80 argocd-sample:local
```

CircleCI が作る `chore: deploy ... [skip ci]` コミットは、イメージタグ更新だけを Argo CD に通知し、CircleCI の再実行を回避します。