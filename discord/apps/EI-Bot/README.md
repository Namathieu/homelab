# EI-Bot

EI-Bot is a Discord bot and local web dashboard for **Empire Immo Monde 8**.
It synchronizes Empire Immo API data into SQLite, analyzes investment opportunities, posts Discord embeds, manages light moderation tools, and exposes the same core tools through a local website.

## Project Context

This project is vibe-coded, meaning it was built with a lot of AI assistance.

The owner is not a professional developer and built this mostly for fun, learning, and helping the Empire Immo community. Expect practical choices, fast iteration, AI-assisted code, and a codebase that favors useful features over formal engineering ceremony.

That said, the project is meant to be understandable and maintainable. If you are another developer testing or improving it, please keep changes simple, documented, and respectful of the current deployment style.

## Main Features

- Discord prefix commands with `!`
- Discord slash commands with `/`
- Local web dashboard bound from `.env` with `WEB_BIND_ADDRESS` and `WEB_PORT`
- Discord-authenticated website with per-server context selection
- SQLite storage for buildings, terrains, materials, factories, players, config, strikes, reaction roles, announcements, and bonus history
- Empire Immo API synchronization with API-key support
- API call guard so normal restarts do not pull the API immediately
- Scheduled data sync at Monde 8 update slots
- Daily historical SQLite backups
- Investment strategy analysis: `AL`, `AV`, `PAV`, `ACL`, `ACV`, `AEL`, `AEV`
- Best affordable investment command
- Company construction estimation
- Player ranking lookup with backup-date support
- Reaction roles
- Strike moderation with role restrictions and GUI flow
- Announcement GUI with role/user/channel targeting
- Daily Entreprise PAV promotion autopost with optional role mention
- Website-only subsidiary bonus calculator

## Runtime Model

The bot is designed to run with Docker Compose.

Services:

- `ei-bot`: Discord bot and scheduler
- `ei-web`: Flask website exposed on `${WEB_BIND_ADDRESS}:${WEB_PORT}`

Mounted runtime folders:

- `database/`: live SQLite database
- `logs/`: runtime logs
- `backups/`: daily and manual SQLite backups

These folders contain local runtime data and are ignored by git and Docker build context.

## Production Environment

`/home/nama/apps/EI-Bot` is the only local runtime. It contains the official bot,
website, live SQLite database, backups, logs, and scheduled Empire Immo synchronization.

Production environment controls:

```env
APP_ENV=production
COMPOSE_PROJECT_NAME=ei-bot-prod
EI_BOT_CONTAINER_NAME=ei-bot
EI_WEB_CONTAINER_NAME=ei-web
WEB_BIND_ADDRESS=192.168.2.250
WEB_PORT=8080
EMPIRE_IMMO_SYNC_ENABLED=true
SCHEDULER_ENABLED=true
```

Useful scripts:

```bash
scripts/prod-test.sh
scripts/prod-deploy.sh
```

## Environment

Create a `.env` file at the project root.

```env
DISCORD_TOKEN=your_discord_bot_token
EMPIRE_IMMO_API_KEY=your_empire_immo_api_key

BUILDINGS_API=https://monde8.empireimmo.com/api/buildings.json
MATERIALS_API=https://monde8.empireimmo.com/api/materials.json
PLAYERS_API=https://monde8.empireimmo.com/api/players.json

GAME_TIMEZONE=Europe/Paris
DAILY_PROMO_TIME=04:01
DAILY_PROMO_TIMEZONE=Europe/Paris

WEB_BIND_ADDRESS=127.0.0.1
WEB_PORT=8080

DISCORD_CLIENT_ID=your_discord_application_client_id
DISCORD_CLIENT_SECRET=your_discord_application_client_secret
DISCORD_REDIRECT_URI=http://127.0.0.1:8080/auth/discord/callback
FLASK_SECRET_KEY=replace_with_a_long_random_secret
WEB_TOKEN_ENCRYPTION_KEY=replace_with_a_long_random_secret
WEB_COOKIE_SECURE=false
```

Optional legacy/default values:

```env
DISCORD_PROMO_CHANNEL_ID=optional_default_channel_id
DISCORD_PROMO_ROLE_ID=optional_default_role_id
```

Important:

- `.env` is ignored and must never be committed.
- Use `.env.example` as the public template for local setup.
- API keys are appended by the client as `?key=<EMPIRE_IMMO_API_KEY>`.
- The bot is configured to avoid API calls on simple restarts when data is already fresh.
- Discord OAuth secrets are backend-only. Never put the client secret, bot token, API key, or token encryption key in frontend code.

## Docker Usage

Start or rebuild everything:

```bash
docker compose up -d --build
```

View logs:

```bash
docker compose logs -f
```

Stop:

```bash
docker compose down
```

Check running services:

```bash
docker compose ps
```

Run tests in Docker:

```bash
docker compose build ei-bot
docker compose run --rm -e PYTHONDONTWRITEBYTECODE=1 ei-bot python -B -m unittest discover -s tests -q
```

## Local Python Usage

Docker is preferred, but local Python can be used for quick development.

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Run one manual sync:

```bash
python -m app.scheduler --once
```

Run the bot:

```bash
python -m app.bot
```

Run the web dashboard:

```bash
flask --app app.web run --host 0.0.0.0 --port 8080
```

## Web Dashboard

The website is served by `ei-web` using the bind address configured in `.env`:

```text
http://<WEB_BIND_ADDRESS>:<WEB_PORT>
```

For local-only testing, use `WEB_BIND_ADDRESS=127.0.0.1`.
For LAN access, set `WEB_BIND_ADDRESS` to the server's LAN IP.

The dashboard includes:

- database summary
- table views
- PAV opportunities
- calculator tools
- paste-table helper
- command-style views for status, player, top, building, strategies, best, and company estimation
- manual sync and backup actions
- subsidiary bonus calculator and saved bonus history

Website authentication:

- users log in with Discord OAuth2
- required scopes: `identify`, `guilds`, `guilds.members.read`
- the browser receives only opaque cookies and safe session data
- OAuth tokens are stored server-side and encrypted in SQLite
- every protected tool requires a selected Discord server
- the server selector lists only guilds where both the user and EI-Bot are present
- browser-provided `guild_id` values cannot switch context and must match the selected server
- state-changing API calls require the CSRF token returned by `/api/session`

Website auth routes:

```text
GET /auth/discord/login
GET /auth/discord/callback
POST /auth/logout
GET /api/session
GET /api/guilds
POST /api/guilds/select
```

Local HTTP warning:

- the LAN dashboard does not provide HTTPS transport security by itself
- public exposure requires HTTPS through a reverse proxy or production web server
- configure the exact `DISCORD_REDIRECT_URI` in the Discord Developer Portal

Website-only feature:

- `Primes Filiales`: paste the benefits table and the directors table, ignore excluded names like `Namathieu`, calculate 35% bonus distribution, cap by `Prime Max`, and generate copy-paste import text.
- bonus history is scoped to the selected Discord server for new records; legacy rows without `guild_id` remain unassigned.

## API Sync Behavior

The scheduler uses `GAME_TIMEZONE`, defaulting to:

```text
Europe/Paris
```

Normal sync runs at:

- `00:00`
- `04:00`
- `08:00`
- `12:00`
- `16:00`
- `20:00`

There is also a final daily pull and backup at:

```text
02:55 Europe/Paris
```

This final pull captures the end-of-day state before the in-game `03:00` update.

API safety:

- normal restarts do not force a new API pull
- `!sync` and `/sync` skip the API when current data is already fresh
- `!sync force` and `/sync force:true` bypass the guard and require Discord administrator permissions

## Backups

Daily backups are written to:

```text
backups/
```

Backup names use the game day that is ending:

```text
ei_bot_YYYYMMDD.db
```

Manual backup commands:

```text
!backup
/backup
```

Manual backups include `manual` in the filename.

## Per-Server Configuration

Discord configuration is intended to be server-specific.

Server-scoped features:

- reaction roles
- strike settings and strike records
- announcement permissions
- daily PAV autopromo channel, role mention, enabled status, and time
- website selected-server context
- website bonus history for new records

The bot maintains a `bot_guilds` inventory from Discord Gateway events. The website intersects that inventory with the authenticated user's OAuth guild list before a server can be selected.

This matters because the same bot can be installed in multiple Discord servers.

## Discord Commands

Most commands exist as both prefix commands and slash commands.

General:

```text
!help
!ping
!status
!sync
!sync force
!backup
```

Slash equivalents:

```text
/help
/ping
/status
/sync
/backup
```

Buildings and players:

```text
!building <building_name>
!player <nickname> [YYYY-MM-DD]
!top10
!top [count] [YYYY-MM-DD]
```

Slash equivalents:

```text
/building
/player
/top10
/top
```

Strategies:

```text
!pav [category] [count]
!al [category] [count] [insurance]
!av [category] [count]
!acl [category] [count] [insurance]
!acv [category] [count]
!ael [category] [count] [insurance]
!aev [category] [count]
```

Slash equivalents:

```text
/pav
/al
/av
/acl
/acv
/ael
/aev
```

Category shortcuts:

```text
p = Perso
e = Entreprise
```

Examples:

```text
!pav
!pav e
!pav e 25
!acl p 10
!acl e 10 prestige
!acv 25 e
```

Default result count is usually `5` per category. Maximum accepted count is `50`.

Insurance choices for rental strategies `AL`, `ACL`, and `AEL`:

```text
tiers
classique
serenite
confort_plus
prestige
rapide_plus
premium
```

Best investment:

```text
!best <p|e> <vente|location> <budget>
/best
```

Example:

```text
!best e vente 25G
```

Company construction:

```text
!company <operation> <category> <building_or_jobs>
/company
```

Operations:

```text
construction
embellishment
repair
renovation
demolition
```

## Daily PAV Autopromo

The bot can post the best current `Entreprise` PAV promotions once per day.

Prefix command:

```text
!autopromo status
!autopromo on [channel_id]
!autopromo off
!autopromo channel <channel_id>
!autopromo role <role_id>
!autopromo roleclear
!autopromo time <HH:MM>
!autopromo test
```

Slash command:

```text
/autopromo
```

Default behavior:

- posts at `04:01 Europe/Paris`
- sends only `Entreprise` PAV promotions
- can mention one configured role
- config is stored per Discord server

## Reaction Roles

Reaction roles give a Discord role when a player reacts to a configured message with a configured emoji.

Prefix commands:

```text
!reactionrole status
!reactionrole set <channel_id> <message_id> <emoji> <role_id>
!reactionrole clear <message_id>
```

Alias:

```text
!rr status
!rr set <channel_id> <message_id> <emoji> <role_id>
!rr clear <message_id>
```

Slash command:

```text
/reactionrole
```

Example:

```text
!rr set 1524040815888826480 1524056858099449990 ✅ 1524039868248756244
```

The bot needs:

- permission to manage roles
- its highest role above the role it gives
- access to the target channel/message

## Strikes

Strikes are moderation records stored per Discord server.

Configure allowed roles:

```text
!setstrike status
!setstrike on <role_id> [role_id ...]
!setstrike off
/setstrike
```

Use strikes:

```text
!strike @player [reason]
!strikestatus
!strikeremove <strike_id>
!mystrikes
```

Slash commands:

```text
/strike
/strikestatus
/strikeremove
/mystrikes
```

`/strike` opens a click panel. From there, moderators can:

- select a player
- add a strike
- choose a channel visible to the target user
- enter a reason
- remove strikes
- view strike status

Regular users can use `/mystrikes` or `!mystrikes` to see only their own strikes.

## Announcements

Announcements are server-scoped and role-restricted.

Configure who can announce:

```text
!setannounce status
!setannounce on <role_id> [role_id ...]
!setannounce off
/setannounce
```

Open the announcement GUI:

```text
!announce
/announce
```

The panel lets the announcer choose:

- message content
- ping or no ping
- target users
- target roles
- one or more channels

Discord administrators can configure announcements. When enabled, only configured roles can use the announcement panel unless the user is an administrator.

## Architecture

The project now uses a layered architecture. For the detailed developer guide, read:

```text
docs/architecture.md
```

The original audit and migration notes remain in:

```text
docs/refactor-audit.md
```

Main folders:

```text
app/core/                       shared permissions and cross-cutting helpers
app/application/user/           player-facing application services
app/application/shareholder/    shareholder operation services
app/application/administration/ server/bot administration services
app/interfaces/discord/         Discord commands, UI, events, presenters
app/interfaces/web/             Flask app, routes package, serializers, static assets
app/infrastructure/persistence/ SQLite repositories, settings, backups, history readers
app/infrastructure/empire_immo/ Empire Immo API client and importers
app/infrastructure/scheduling/  scheduler setup, sync, and jobs
tests/                          unittest test suite
```

Important files:

```text
app/bot.py        Discord bot bootstrap and slash sync
app/scheduler.py  stable scheduler CLI wrapper
app/web.py        stable Flask import wrapper
app/autopromo.py  compatibility facade for autopromo config/jobs
app/reaction_roles.py
app/strikes.py
app/announce.py
app/bonus.py      website-only subsidiary bonus calculator
```

Compatibility note:

- `app.analysis.*`, `app.commands.*`, `app.api.*`, and `app.database.*` still exist as compatibility modules.
- New code should prefer `app/application`, `app/interfaces`, and `app/infrastructure`.
- Runtime data remains in root-level `database/`, `backups/`, and `logs/`.

## Refactor Direction

Future development should move the project toward three functional domains while keeping one Discord bot identity:

- `User`: normal player-facing tools such as strategies, building lookup, player lookup, rankings, best investments, company estimates, calculators, status, and `mystrikes`.
- `Shareholder`: shareholder operational tools such as strike creation, strike status, strike removal, and subsidiary bonus calculations.
- `Administration`: bot and Discord management such as announcements, reaction roles, autopromo, sync, backups, strike-role configuration, and server-specific configuration.

The active layout is:

```text
app/
    core/                  shared config, permissions, errors, formatting
    application/
        user/              player-facing application services
        shareholder/       shareholder operation services
        administration/    bot/server administration services
    interfaces/
        discord/           commands, events, UI, presenters
        web/               Flask app, routes package, serializers, static assets
    infrastructure/
        persistence/       SQLite repositories, settings, backups
        empire_immo/       Empire Immo API client and importers
        scheduling/        scheduled jobs and sync orchestration
```

Do not split this into multiple Discord bots. The bot and website may remain separate runtime services, but there must only be one Discord bot token and identity.

Dependency rules:

- Discord and web adapters should call domain services.
- Scheduled jobs should call application/domain services.
- Domain services may call persistence repositories and external integrations through clear boundaries.
- Business logic must not import Discord UI classes or Flask routes.
- Persistence must not import command handlers, Discord, or Flask.
- Runtime data must stay in root-level `database/`, `backups/`, and `logs/`.

## Permission Model

Permissions should be capability-based, not treated as one automatic hierarchy.

Current capabilities:

- `public user`: default access for normal player-facing commands
- `shareholder`: access to shareholder operations such as strike commands
- `Discord administrator`: access to server/bot configuration commands

Important rule:

- A Discord administrator must not automatically receive shareholder permissions.
- A shareholder must not automatically receive Discord administrator permissions.
- A person may have both capabilities if configured that way.

Current strike behavior already follows the important shareholder/admin separation: admins configure the strike roles, but only configured strike roles can issue, view, or remove strikes.

## Language Rules

Developer communication, architecture docs, implementation plans, comments, internal identifiers, filenames, classes, and functions may be in English.

Discord-facing content should be French:

- command names and descriptions where practical
- command responses
- embeds
- errors and permission-denied messages
- help text
- buttons, selects, modals, labels, confirmations, warnings
- automated Discord messages

The Discord adapter layer has a regression test guarding high-visibility English phrases that were translated during the refactor.

## Development Rules

When changing the project:

- preserve existing behavior unless the change was explicitly requested
- do not add new feature systems during structural refactors
- keep Discord prefix and slash commands complete and unique
- keep configuration server-specific when it involves Discord guild roles, channels, messages, or moderation state
- do not move generated DB files, backups, or logs into `app/`
- do not delete or rewrite historical backups
- do not commit `.env`, live DB files, logs, or secrets
- prefer adding focused regression tests before moving risky modules
- run syntax checks and the Docker unittest suite before considering work complete
- use `PYTHONDONTWRITEBYTECODE=1` and `python -B` for validation runs so generated Python caches are not left behind

## Validation

Run the automated tests:

```bash
docker compose build ei-bot
docker compose run --rm -e PYTHONDONTWRITEBYTECODE=1 ei-bot python -B -m unittest discover -s tests -q
```

Syntax check without bytecode:

```bash
PYTHONDONTWRITEBYTECODE=1 python3 -B - <<'PY'
import ast
from pathlib import Path
for path in sorted(Path("app").rglob("*.py")) + sorted(Path("tests").rglob("*.py")):
    ast.parse(path.read_text(encoding="utf-8"), filename=str(path))
PY
```

Manual Discord validation checklist:

```text
VALIDATION.md
```

Useful runtime checks:

```bash
docker compose ps
docker compose logs --tail=100 ei-bot
docker compose logs --tail=100 ei-web
curl -fsS -o /dev/null -w '%{http_code}\n' http://<WEB_BIND_ADDRESS>:<WEB_PORT>/
```

## Logs

Runtime logs are written to:

```text
logs/ei-bot.log
```

Logs include:

- bot startup
- Discord login
- slash command sync
- command errors
- scheduler start
- sync decisions
- API import failures
- backup creation

## Git-Ignored Local Data

The following are intentionally ignored:

```text
.env
venv/
database/*.db
database/*.db-*
backups/*.db
backups/*.tmp
logs/*
*.log
__pycache__/
*.py[cod]
.pytest_cache/
.coverage
htmlcov/
```

Do not commit secrets, live databases, logs, or generated Python caches.

## Version

Current version is stored in:

```text
VERSION
```

The version is displayed by:

```text
!status
/status
```

## Known Issues and Limitations

- The website currently runs with Flask's development server inside Docker.
- SQLite is enough for the current Discord volume, but a heavier deployment may eventually need a server database.
- Some formulas are practical estimates based on observed game behavior and may need calibration over time.
- Fuzzy matching uses Python standard-library similarity, so unusual typos may miss suggestions.
- Discord screenshots and permission behavior must still be validated manually in the target server.
