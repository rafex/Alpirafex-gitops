# Operación

## Primer release

Antes de reconciliar la aplicación:

1. Configura en `rafex/Alpirafex` estos Secrets de GitHub Actions:
   `ALPIRAFEX_PACKAGER_PRIVKEY`, `ALPIRAFEX_PACKAGER_PUBKEY` y
   `ALPIRAFEX_GITOPS_TOKEN`.
2. Publica como públicos los paquetes `alpirafex-web` y
   `alpirafex-repository` en GHCR.
3. Ejecuta el workflow de la landing y el workflow de release APK.
4. Fusiona los PRs de promoción de digest.
5. Comprueba que ya no queden referencias `sha256:000...`:

   ```sh
   rg 'sha256:0{64}' clusters
   ```

Los endpoints publicados son:

- `https://alpirafex.rafex.io/` para la landing.
- `https://alpirafex.rafex.io/alpirafex/v3.24/x86_64/` para x86_64.
- `https://alpirafex.rafex.io/alpirafex/v3.24/aarch64/` para aarch64.

El registro DNS `alpirafex.rafex.io` debe apuntar a `158.69.246.55`. La
terminación TLS y la renovación del certificado las gestiona
`k3s-haproxy-setup` al detectar los Ingress; no se guarda un Secret TLS en
este repositorio.

## Flux en Server 2

La autenticación SSH de Flux vive únicamente en el Secret
`flux-system/alpirafex-gitops-auth`. La aplicación permanece con `prune: false`
durante la primera validación.

```sh
export KUBECONFIG="$HOME/.kube/config_k3s_server2"
flux reconcile source git flux-system
flux reconcile kustomization flux-system --with-source
flux reconcile source git alpirafex-gitops
flux reconcile kustomization alpirafex --with-source
flux get sources git -A
flux get kustomizations -A
kubectl -n alpirafex-web get deploy,pods,svc,ingress
kubectl -n alpirafex-repo get deploy,pods,svc,ingress
```

Después de validar ambos endpoints y los dos índices APK, cambia `prune` a
`true` mediante un PR en `k3s-gitops`.
