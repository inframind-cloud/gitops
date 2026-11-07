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

Ingress com `ingressClassName: traefik` e host `*.inframind.cloud` recebe DNS pela Cloudflare (external-dns, proxy ligado). O alvo é o IP do Service do Traefik.

## Segredos

Não entram neste repositório. O External Secrets lê o Doppler pelo `ClusterSecretStore` `doppler`. O único Secret criado à mão é `doppler-token` no namespace `external-secrets` (service token, chave `dopplerToken`).

| Secret no cluster | Chave | Nome no Doppler |
| --- | --- | --- |
| `cloudflare-api-token` (`external-dns`) | `apiToken` | `CLOUDFLARE_API_TOKEN` |
| `cloudflare-api-token` (`cert-manager`) | `api-token` | `CLOUDFLARE_API_TOKEN` |
| `traefik-dashboard-auth` (`kube-system`) | `users` | `TRAEFIK_DASHBOARD_USERS` |
| `livekit-keys` (`livekit`) | `keys.yaml` | `LIVEKIT_KEYS` |

Uma app nova ganha um `ExternalSecret` em `apps/<namespace>/<nome>/` a apontar para o mesmo store.

## LiveKit

Sinalização em `wss://livekit.inframind.cloud`. O TURN é `turn.inframind.cloud` (DNS only, registo A criado à mão para o IP da VPS). As apps assinam o JWT com a API key do Secret; a key não vai para o browser.
