# WorkPlanner – Fejlesztői Környezet Dokumentáció

Ez a repository a **WorkPlanner mikroszolgáltatási architektúra** helyi fejlesztői környezetének központi vezérlője.

## 1. Előfeltételek

- **Docker**
- **Git**
- A szolgáltatási repository-k **egy közös gyökérmappában** legyenek klónozva.
- **Portok szabadon**: 3000 (Nuxt), 5432 (PostgreSQL)
- Javasolt `npm i -g @antfu/ni` a gördülékenyebb csomagkezeléshez.

## 2. Mappastruktúra

A következő struktúra szükséges a `docker-compose.yml` helyes működéséhez:

```
system-compose/               <-- Ez a repository
├── README.md
├── docker-compose.yml
├── core-app/                  <-- Nuxt / Full-stack alkalmazás
│   ├── Dockerfile
│   └── (... Nuxt kód)
└── other-service-repo/         <-- Egyéb szolgáltatás (példa)
```

## 3. Indítási Protokoll

### 3.1. Tisztítás

```
docker compose down -v
```

### 3.2. Indítás

```
docker compose up --build
```

## 4. Hozzáférési Pontok

| Szolgáltatás                       | Host/Port             | Funkció                          |
| ---------------------------------- | --------------------- | -------------------------------- |
| **Nuxt App**                       | http://localhost:3000 | Frontend + API végpontok         |
| **PostgreSQL**                     | localhost:5432        | Adatbázis elérés (IDEA, pgAdmin) |
| **PostgreSQL (Nuxt belső kapcs.)** | db:5432               | Adatbázis belső kapcsolat        |

## 5. Adatbázis adatok

**USER:** `user` • **PASS:** `postgrespassword` • **DB:** `workplanner` • **SCHEMA:** `workplanner`

## 5. Fejlesztői Jegyzetek

### 5.1. Stabilitási Logika (Healthcheck)

- `app` csak akkor indul, ha `db` állapota **service_healthy**.

### 5.3. Volume Kezelés

| Volume                         | Funkció                       |
| ------------------------------ | ----------------------------- |
| `workplanner-data`             | Perzisztens PostgreSQL mentés |
| `node_modules_cache`           | Teljesítményoptimalizálás     |
| Kód mount (`../core-app:/app`) | Hot Reloading                 |
