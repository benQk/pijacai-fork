<!-- — opis, tehnologije, zagon in demonstracijski podatki. -->
# Pijacai

Pijacai je spletni sistem za vodenje proizvodnih šarž v manjši polnilnici pijač. Omogoča načrtovanje proizvodnih nalogov, evidentiranje surovin in njihove porabe, kontrolo kakovosti ter sledljivost med serijami surovin in končnimi šaržami.

## Tehnologije

- **Frontend:** React, TypeScript in Vite
- **Backend:** Python in FastAPI
- **Podatkovna baza:** PostgreSQL
- **Dostop do podatkov in migracije:** SQLAlchemy in Alembic
- **Zagon okolja:** Docker Compose


## Lokalni zagon

Potrebuješ zagnan Docker Desktop z Linux kontejnerji in Docker Compose. Ukaze izvajaj v korenski mapi projekta.

Ob prvi postavitvi ustvari lokalno konfiguracijo:

```powershell
Copy-Item .env.example .env
```

Nastavitve prilagodi v `.env`, ki ostane lokalna. Nato zaženi aplikacijo:

```powershell
docker compose up --build
```

- Aplikacija: [http://localhost:5174](http://localhost:5174)
- API dokumentacija: [http://localhost:8001/docs](http://localhost:8001/docs)
- PostgreSQL: `localhost:5433`

Backend pred zagonom samodejno izvede migracije baze. Za prvo prijavo v drugem terminalu ustvari začetnega administratorja:

```powershell
docker compose exec backend python -m app.seed
```

Sledi pozivom za ime, e-pošto in geslo, nato se prijavi v aplikaciji. Obstoječega računa ni treba ustvariti znova.

Za ustavitev uporabi:

```powershell
docker compose down
```

Podatki baze se ohranijo v Dockerjevem volumnu.

## Demonstracijski podatki

V zagnanem okolju pripravi šifrante, izdelek z recepturo, proizvodne linije in parametre kakovosti:

```powershell
docker compose exec backend python -m app.seed_inventory
docker compose exec backend python -m app.seed_production
docker compose exec backend python -m app.seed_lines
docker compose exec backend python -m app.seed_quality
```

Skripte dodajo manjkajoče demonstracijske podatke. Obstoječe aktivne recepture in mej kakovosti ne spreminjajo.

Kot administrator na strani **Uporabniki** ustvari račune za skladiščnika (`warehouse`), planerja (`planner`), operaterja (`operator`) in kontrolorja (`quality`). Prijavi se kot skladiščnik in prek **Prejem surovin** evidentiraj začetno zalogo; skripte zaloge ne ustvarijo.

Osnovni potek: prejem surovin → proizvodni nalog → zagon šarže → zaključek polnjenja → meritve kakovosti → sprostitev ali zavrnitev → sledljivost.

## Dokumentacija

- [Zasnova rešitve](docs/01-zasnova-resitve.md)
- [Funkcionalnosti in obseg](docs/02-funkcionalnosti-in-obseg.md)
- [Podatkovni model](docs/03-podatkovni-model.md)
- [Arhitektura in tehnološki sklad](docs/04-arhitektura-in-tehnoloski-sklad.md)
- [Uporabniške zgodbe](docs/05-uporabniske-zgodbe.md)
- [Wireframes](docs/06-wireframes.md)
- [Arhitekturne odločitve (ADR)](docs/adr/)

## CI/CD

![CI/CD](https://github.com/<uporabnik>/pijacai-fork/actions/workflows/ci-cd.yml/badge.svg)

**Nameščena aplikacija:** [https://pijacai.vercel.app](https://pijacai.vercel.app)
(frontend na Vercelu, backend in PostgreSQL na Renderju; API dokumentacija:
[https://pijacai-fork.onrender.com/docs](https://pijacai-fork.onrender.com/docs))

> Opomba: brezplačni Render strežnik po obdobju nedejavnosti zaspi, zato lahko
> prvi zahtevek traja približno minuto.

### Faze cevovoda

Workflow `.github/workflows/ci-cd.yml` se sproži ob vsakem pull requestu v vejo
`main` in ob vsakem pushu na `main`.

1. **lint** – preveri slog in kakovost kode: ESLint za frontend in Ruff za backend.
   Če linter najde napako, job pade.
2. **build** – zgradi frontend (`npm run build`) in pripravi paket backenda.
   Odvisnosti (npm, pip) se predpomnijo. Rezultata (`frontend/dist` in backend
   paket) se shranita kot artefakta.
3. **test** – zažene teste frontenda (Vitest) in backenda (pytest z
   integracijskimi testi na PostgreSQL servisu). Poročila o pokritosti kode se
   shranijo kot artefakt. Če katerikoli test pade, pade tudi cevovod.
4. **deploy** – teče samo ob pushu na `main` in samo, če so uspešno zaključeni
   jobi `lint`, `build` in `test`. Uporablja okolje `production` in artefakte
   iz joba `build` (aplikacije ne gradi ponovno). Frontend se namesti na
   Vercel, backend pa na Render prek deploy hooka. Skrivnosti so shranjene kot
   GitHub Secrets okolja `production`.

Veja `main` je zaščitena: spremembe se združujejo samo prek pull requestov,
obvezne preverbe pa so `lint`, `build` in `test`. Pull requesta s padajočim
cevovodom zato ni mogoče združiti.