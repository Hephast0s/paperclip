# Self-hosted Kubernetes

`kubernetes-self-hosted.yaml` runs one Paperclip server and one PostgreSQL 17
instance with persistent volumes. It targets a small private cluster with the
`local-path` storage class and Traefik ingress.

Create the private credentials once, then apply the resources:

```sh
kubectl create namespace paperclip
kubectl -n paperclip create secret generic paperclip-secrets \
  --from-literal=postgres-password="$(openssl rand -hex 32)" \
  --from-literal=better-auth-secret="$(openssl rand -hex 48)" \
  --from-literal=tool-action-signing-secret="$(openssl rand -hex 32)"
kubectl apply -f docs/deploy/kubernetes-self-hosted.yaml
```

The default address is `http://paperclip.192.168.1.8.nip.io`; change the host
in the manifest and `PAPERCLIP_PUBLIC_URL` / `PAPERCLIP_ALLOWED_HOSTNAMES` to
match your private DNS or ingress address. The default HTTP ingress is for a
trusted LAN only. For access over the public internet, put authenticated
Paperclip behind HTTPS and update those values to the HTTPS origin.

The deployment requests 1.5 GiB for Paperclip and 512 MiB for PostgreSQL, with
limits of 4 GiB and 1.5 GiB. It runs a single replica of each. Both volumes use
node-local storage, so they persist through pod restarts but are not highly
available and need backups before node or disk replacement. Agent integrations
and provider credentials are configured in Paperclip after first login. The
checked-in manifest pins Paperclip to `worker-1` because that node has working
GHCR/CDN access; change the hostname selector and use a node-local data PVC on
that node when applying it to a different cluster.
