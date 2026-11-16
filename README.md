# Aetheria RPG — Cloudflare Workers build

This repository is a Cloudflare Workers + Static Assets + D1 adaptation of the Aetheria web-game foundation. It keeps the WhatsApp bot deployment separate.

## Included and working in this build

- Responsive Aetheria web interface (the existing web UI is in `public/`).
- Email/password registration and login; password hashes use PBKDF2-SHA-256 and sessions are stored as SHA-256 token hashes.
- Persistent player profiles and progress in Cloudflare D1.
- 100 generated hunt floors, server-side battle calculations, XP and resource rewards.
- Weapon, armour and artifact gacha (1, 10 and 50 pulls).
- Inventory equip/unequip/sell.
- Beast summoning, training and release.
- Skill selection (up to five equipped skills).
- Guild creation and joining.
- Leaderboard and daily rewards.

## Important scope and production notes

This is a deployable Cloudflare conversion of the web-game features listed above, not a claim that every WhatsApp command has already been ported. Full guild raids/contribution payouts, player-to-player marketplace/trading, PvP, Beast Arena tournament scheduling, beast breeding/evolution, advanced class-rank skill progression, spellbook drops, admin tools, email verification, password reset, abuse/rate limiting, and import/linking of existing WhatsApp player saves still need dedicated implementation and testing. The original WhatsApp source is retained in `original-whatsapp-source/` for reference; it is not executed by the Cloudflare Worker.

New web accounts are independent IDs prefixed with `web_`. No existing WhatsApp player database is included or migrated by this package. Back up and design an explicit account-linking/migration process before combining accounts.

Before opening public registration, add rate limiting/Turnstile, review abuse controls, configure backups, and test with multiple accounts. Never commit passwords, API tokens, or production secrets.

## Cloudflare setup

### 1. Create the D1 database

In Cloudflare Dashboard:
1. Open **Workers & Pages → D1 SQL Database** (or **Storage & databases → D1**).
2. Create a database named `aetheria-rpg-db`.
3. Open its **Console** and run the complete SQL from `migrations/0001_init.sql`.
4. Copy the database ID shown in the database overview.

### 2. Connect this repository to a Worker

1. Put the contents of this project (the files inside this folder, not the enclosing ZIP) in the root of a GitHub repository named `aetheria-rpg`.
2. In Cloudflare, choose **Workers & Pages → Create application → Workers → Connect to Git** and select that repository.
3. Project/Worker name: `aetheria-rpg`.
4. Root directory/path: `/`.
5. Build command: leave blank.
6. Deploy command: `npx wrangler deploy`.
7. Preview command: leave blank or use `npx wrangler dev` only for preview if prompted.

### 3. Bind the database

Edit `wrangler.jsonc` and replace `REPLACE_WITH_D1_DATABASE_ID` with the actual D1 database ID. Keep the binding name exactly `DB` and the database name exactly `aetheria-rpg-db`. Commit the change to GitHub so Cloudflare redeploys.

The static files are served from `public/` through the `ASSETS` binding configured in `wrangler.jsonc`.

### 4. Deploy

The repository has `package.json` with Wrangler as a development dependency. Cloudflare's build should install dependencies and run `npx wrangler deploy`. If the first deploy happens before the D1 ID is filled in, it will fail; update the ID and redeploy.

The resulting site will be available at the Worker `*.workers.dev` address shown in Cloudflare.

## Local development (requires Node.js 20+ and internet to install Wrangler)

```bash
npm install
npx wrangler login
npx wrangler d1 create aetheria-rpg-db
# Copy the returned database_id into wrangler.jsonc
npx wrangler d1 execute aetheria-rpg-db --local --file=migrations/0001_init.sql
npm run dev
```

For production D1, apply the SQL remotely before deploying:

```bash
npx wrangler d1 execute aetheria-rpg-db --remote --file=migrations/0001_init.sql
npm run deploy
```

## Repository layout

- `src/index.js` — Cloudflare Worker API and game logic.
- `public/` — browser game UI and beast artwork.
- `migrations/0001_init.sql` — D1 schema.
- `wrangler.jsonc` — Worker, static assets and D1 binding configuration.
- `original-whatsapp-source/` — preserved source reference from the uploaded WhatsApp RPG ZIP; not imported into the Worker runtime.
