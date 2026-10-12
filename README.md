# Modus SEBI POC — container image

`modus-app:1.0` — one container: nginx (8080) in front of Next.js (3000) and FastAPI (8000).
Built for **linux/amd64**. Works with either docker or podman.

The image is split into 90 MB parts under `parts/` because git rejects files over 100 MB.

## Restore

```bash
# 1. reassemble (order matters; the glob sorts correctly)
cat parts/modus-app-1.0.tar.gz.* > modus-app-1.0.tar.gz

# 2. verify — MUST print OK before continuing
sha256sum -c SHA256SUMS.txt

# 3. load
docker load -i modus-app-1.0.tar.gz      # or: podman load -i ...
```

## Run

```bash
docker run -d --name modus-app -p 8080:8080 \
  -e DEPLOY_ENV=onprem \
  -e SSH_TUNNEL_ENABLED=false \
  -e DB_HOST=<db-vm-ip> -e DB_PORT=5432 \
  -e DB_NAME=<db> -e DB_USER=<user> -e DB_PASSWORD=<pw> \
  modus-app:1.0
```

Check it:

```bash
curl -I http://localhost:8080/          # UI        -> 200
curl -I http://localhost:8080/docs      # API docs  -> 200
docker logs modus-app                   # all three processes, one stream
```

With podman add `:Z` to any volume mount, or SELinux blocks it.

## If the ZIP is too large to download

Each part is under 100 MB, so they can be fetched one at a time instead:

```bash
for p in aa ab ac ad ae af ag ah ai aj ak al am an ao ap; do
  curl -LO https://raw.githubusercontent.com/ananya-modusai/sebi-ui/main/parts/modus-app-1.0.tar.gz.$p
done
cat modus-app-1.0.tar.gz.* > modus-app-1.0.tar.gz
```
