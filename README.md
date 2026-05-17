# 2D Survival Shooter

Top-down 2D arena shooter v jediném `index.html` (vanilla JS + Canvas, žádné dependencies).
Autor: **Jonáš Soldát**.

## Hra

- **Ovládání**: WASD nebo šipky = pohyb, myš = míření, mezerník/klik = útok, **E** = přepnout zbraň, **Shift** = uvolnit kurzor
- **Zbraně**: pistole, baseballová pálka, AK47 (odemkne se na 30 killech), RPG (200 killů)
- **Vlny**: 5 vln s novými typy nepřátel, +50 % spawn rate na vlnu
- **Bossové**: BOSS na 900 bodech (25k HP), FINAL BOSS na 2500 bodech (2,5M HP) — jediná cesta k vítězství
- **Upgrade menu**: před každým bossem (na 899 a 2499 bodech) výběr z 7 stupňů vzácnosti, 23+ upgradů
- **Módy**: Survival, Sandbox (`T` = přidat skóre, `U` = volný upgrade)
- **Endless** po poražení Final Bosse — scaling HP/rychlosti/spawn rate

## Lokální spuštění

Vyžaduje Docker. `make` (bez argumentů) postaví image, spustí Traefik + nginx a hra běží na **http://2d.localhost**.

```bash
make            # build + start (default target)
make logs       # tail logů
make down       # zastavit a odstranit
make help       # všechny targety
```

Traefik dashboard: http://localhost:8080.

## Deployment

Cílový server: **primus.nadoma.net** (Docker Swarm, externí Traefik), doména **https://2d.nadoma.net**, registry **registry.nadoma.net**.

Předpoklady (jednorázově):

```bash
docker login registry.nadoma.net          # lokálně
ssh admin@primus.nadoma.net docker login registry.nadoma.net   # na serveru
```

Deploy:

```bash
make deploy     # buildx amd64 push → ship resolved stack → docker stack deploy
```

`make deploy`:

1. `docker buildx build --platform linux/amd64 --push` — cross-build z Macu a push do registry s tagy `<commit-hash>` a `latest`
2. `docker compose config` lokálně vyrenderuje a vysubstituuje `docker-compose.yaml` + `docker-compose.deploy.yaml` do hotového stack souboru
3. Stack soubor poslán přes SSH na `/srv/2D-game/docker-stack.yaml`
4. `docker stack deploy --with-registry-auth --prune` na primusu

## Struktura

```
index.html                   hra (HTML + JS + Canvas)
Dockerfile                   nginx:alpine + index.html
docker-compose.yaml          base — service + Traefik labels
docker-compose.dev.yaml      local — Traefik kontejner + build context
docker-compose.deploy.yaml   prod — externí dmz síť + Swarm deploy
Makefile                     targety pro dev i deploy
```
