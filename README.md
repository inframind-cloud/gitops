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

Não entram neste repositório. O Dopbase corre no cluster e a UI é `https://dopbase.inframind.cloud`. Os Secrets que o Traefik, o external-dns, o cert-manager e o LiveKit já usam continuam no cluster. A masterkey do ZITADEL está no Secret `zitadel-masterkey` e a password inicial do admin no Secret `zitadel-admin-config`, ambos no namespace `zitadel`.

## ZITADEL

Consola em `https://auth.inframind.cloud`. O primeiro utilizador é `renato`.

## LiveKit

Sinalização em `wss://livekit.inframind.cloud`. O TURN é `turn.inframind.cloud` (DNS only, registo A criado à mão para o IP da VPS). As apps assinam o JWT com a API key do Secret; a key não vai para o browser.
