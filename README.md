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

Não entram neste repositório. O token da Cloudflare está no Secret `cloudflare-api-token` (`external-dns`). O htpasswd do dashboard está no Secret `traefik-dashboard-auth` (`kube-system`). A API key do LiveKit está no Secret `livekit-keys` (`livekit`).

## LiveKit

Sinalização em `wss://livekit.inframind.cloud`. O TURN é `turn.inframind.cloud` (DNS only, registo A criado à mão para o IP da VPS). As apps assinam o JWT com a API key do Secret; a key não vai para o browser.
