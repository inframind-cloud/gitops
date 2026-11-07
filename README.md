# gitops

Estado do cluster `local-k3s` (VPS `tim-cloud`). O Argo CD aplica esta árvore.

```
bootstrap/          AppProject, ApplicationSets e o root app
namespaces/<nome>/  Namespace com labels. Uma pasta = uma Application ns-<nome>
platform/<nome>/    Addons (Traefik, Argo CD). Uma pasta = uma Application
apps/<ns>/<app>/   Workloads. A Application chama-se <ns>-<app> e corre em <ns>
```

O root app (`bootstrap/`) é o único objeto aplicado à mão. O resto entra por ApplicationSet.

## Acrescentar um namespace

Cria `namespaces/<nome>/namespace.yaml` e faz push. O ApplicationSet `namespaces` sincroniza.

## Acrescentar uma app

Copia `apps/_template` para `apps/<namespace>/<app>`. O `_template` está excluído do generator.

Ingress com `ingressClassName: traefik` e host `*.inframind.cloud` ou `*.sidespace.chat` recebe DNS pela Cloudflare (external-dns, proxy ligado). O alvo é o IP do Service do Traefik.

## SideSpace

Namespace `sidespace`. A UI fica em `https://app.sidespace.chat` e a API em `https://api.sidespace.chat`. As variáveis estão no Dopbase, projeto `sidespace`, ambiente `production`.

Deploy: um push em `master` (backend) ou `main` (web) gera uma imagem amd64 no GHCR. O CI faz commit da tag `sha-<commit>` em `apps/sidespace/{api,web}/kustomization.yaml`, e o Argo CD sincroniza. A API corre as migrações ao arrancar (`SIDESPACE_AUTO_MIGRATE=true`).

O External Secrets lê o ambiente pelo SecretStore `dopbase` (provider webhook, `apps/sidespace/secrets`). Gera dois Secrets: `sidespace-api-env`, com todas as variáveis menos `GHCR_*`, e `ghcr-pull`, a partir de `GHCR_PULL_USER` e `GHCR_PULL_TOKEN`. O repositório gitops não aceita deploy keys. O CI autentica com o secret `GITOPS_TOKEN` (PAT com escrita neste repositório). As imagens no GHCR são privadas: o ExternalSecret `ghcr-pull` usa `GHCR_PULL_USER` e `GHCR_PULL_TOKEN` (`read:packages`).

O único Secret criado à mão no cluster é o runner token:

```sh
kubectl -n sidespace create secret generic dopbase-runner --from-literal=token=<runner token>
kubectl -n sidespace label secret dopbase-runner external-secrets.io/type=webhook
```

O ExternalSecret atualiza o Secret de hora a hora, mas um Secret alterado não reinicia os pods. Depois de mudar uma variável: `kubectl -n sidespace rollout restart deploy/sidespace-api`.

## Segredos

Não entram neste repositório. O Dopbase corre no cluster e a UI é `https://dopbase.inframind.cloud`. Os Secrets que o Traefik, o external-dns, o cert-manager e o LiveKit já usam continuam no cluster.

## LiveKit

Sinalização em `wss://livekit.inframind.cloud`. O TURN é `turn.inframind.cloud` (DNS only, registo A criado à mão para o IP da VPS). As apps assinam o JWT com a API key do Secret; a key não vai para o browser.
