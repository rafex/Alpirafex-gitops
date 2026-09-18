# Alpirafex GitOps

Manifiestos declarativos para publicar Alpirafex en el cluster K3s `server2`.

## Dominios

- `https://alpirafex.rafex.io`: landing estática.
- `https://alpirafex.rafex.io/alpirafex/v3.24/`: repositorio APK firmado.

La terminación TLS, DNS y HAProxy son responsabilidad del host y de
`k3s-haproxy-setup`. Kubernetes recibe tráfico HTTP mediante la `IngressClass`
`haproxy`; la terminación TLS se realiza en HAProxy.

## Estructura

- `base/web`: landing y namespace `alpirafex-web`.
- `base/repository`: repositorio APK y namespace `alpirafex-repo`.
- `clusters/server2`: composición del despliegue para Server 2.
- `site`: fuente de la imagen pública `ghcr.io/rafex/alpirafex-web`.

El repositorio APK usa la misma autoridad HTTP que la web, bajo el prefijo
`/alpirafex/v3.24/`. Así, una instalación Alpine puede usar:

```text
https://alpirafex.rafex.io/alpirafex/v3.24/<arquitectura>/
```

Las imágenes se fijan por digest en los overlays. Los workflows publican las
imágenes y abren PRs para actualizar esos digests.

## Validación local

```sh
kubectl kustomize clusters/server2/web
kubectl kustomize clusters/server2/repository
kubectl kustomize clusters/server2
```

El primer release debe reemplazar los digests `sha256:000...` mediante los PRs
generados por CI antes de reconciliar la aplicación con Flux. El DNS público de
`alpirafex.rafex.io` debe apuntar a `158.69.246.55` para que
`k3s-haproxy-setup` pueda emitir y renovar el certificado TLS.
