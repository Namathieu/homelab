# EI-Bot V1 Validation Checklist

Use this checklist during Discord server validation. Mark each item after testing in a real channel where EI-Bot can read messages and send embeds.

## Command Checklist

For each command, verify:

- command executes
- no unhandled exception appears in `logs/ei-bot.log`
- embed or message renders correctly
- formatting is readable on desktop and mobile
- pagination works when requesting many results

Commands:

- [ ] `!ping`
- [ ] `!status`
- [ ] `!building Grande maison équipée`
- [ ] `!building "Grande maison équipée"`
- [ ] `!player Math_vis`
- [ ] `!top10`
- [ ] `!pav`
- [ ] `!pav 25`
- [ ] `!al`
- [ ] `!al 50`
- [ ] `!av`
- [ ] `!acl`
- [ ] `!acv`
- [ ] `!ael`
- [ ] `!aev`

## Building Lookup Tests

- [ ] Exact name: `!building Grande maison équipée`
- [ ] Quoted exact name: `!building "Grande maison équipée"`
- [ ] Partial name: `!building atelier`
- [ ] Misspelled name: `!building Grande maison equippe`
- [ ] Unknown name: `!building impossible-building-name`

Expected result: exact matches display full analysis; partial or misspelled names return useful suggestions; unknown names fail gracefully.

## Player Lookup Tests

- [ ] Top player: `!player patrick`
- [ ] Middle-ranked player: choose a nickname outside the top 100
- [ ] Requested sample: `!player Math_vis`
- [ ] Partial nickname
- [ ] Misspelled nickname
- [ ] Nonexistent nickname

Expected result: exact matches display player details; partial or misspelled names return suggestions; unknown names fail gracefully.

## Strategy Review

- [ ] AL ranks by rental ROI
- [ ] AV ranks resale opportunities by ROI and profit
- [ ] PAV includes only promotions at or above 8% and sorts by value descending
- [ ] ACL includes terrain plus construction cost and rental net income
- [ ] ACV includes terrain, construction, taxes, ROI, and profit per day when duration data exists
- [ ] AEL maps current building to next embellishment tier
- [ ] AEV maps current building to next embellishment tier

If a ranking looks suspicious, record the command, displayed row, and expected behavior before changing formulas.

## Scheduler Validation

Confirm in `logs/ei-bot.log` that sync starts and completes at:

- [ ] 00:00 Europe/Paris
- [ ] 04:00 Europe/Paris
- [ ] 08:00 Europe/Paris
- [ ] 12:00 Europe/Paris
- [ ] 16:00 Europe/Paris
- [ ] 20:00 Europe/Paris
- [ ] final daily pull and backup at 02:55 Europe/Paris

Manual scheduler validation:

```bash
python -m app.scheduler --once
```

Expected result: imports complete and `Scheduler sync completed` appears in `logs/ei-bot.log`.

Guardrail: a normal restart should not force a fresh Empire Immo API pull if the current data is already fresh.

## Performance Validation

Run multiple commands quickly from Discord:

- [ ] `!status`
- [ ] `!building Grande maison équipée`
- [ ] `!player Math_vis`
- [ ] `!pav 25`
- [ ] `!al 50`
- [ ] `!acv 50`

Expected result: no crashes, no SQLite lock errors, no scheduler conflicts.

## Production Validation

- [ ] production worktree is `/home/nama/apps/EI-Bot`
- [ ] branch is `master`
- [ ] working tree is clean
- [ ] `.env` contains `APP_ENV=production`
- [ ] `.env` contains `COMPOSE_PROJECT_NAME=ei-bot-prod`
- [ ] `.env` contains `WEB_PORT=8080`
- [ ] `.env` contains `EMPIRE_IMMO_SYNC_ENABLED=true`
- [ ] `.env` contains `SCHEDULER_ENABLED=true`
- [ ] `scripts/prod-test.sh`
- [ ] `scripts/prod-deploy.sh`
- [ ] bot starts
- [ ] bot connects to Discord
- [ ] scheduler starts
- [ ] database counts are logged
- [ ] next synchronization time is logged
- [ ] command execution is logged
- [ ] `docker restart ei-bot` recovers cleanly
- [ ] VM reboot recovery confirmed after Docker starts on boot

## Website OAuth Validation

- [ ] Discord Developer Portal redirect URI matches `DISCORD_REDIRECT_URI`
- [ ] OAuth scopes include `identify`, `guilds`, and `guilds.members.read`
- [ ] `/auth/discord/login` redirects to Discord authorization
- [ ] callback creates a local server-side session
- [ ] `/api/session` does not expose OAuth tokens
- [ ] server selector shows only shared guilds where EI-Bot is installed
- [ ] switching guilds clears previous results
- [ ] protected tools are disabled without a selected guild
- [ ] state-changing requests fail without `X-CSRF-Token`
- [ ] sync and backup are clearly treated as global actions authorized by selected-guild admin permission
- [ ] shareholder tools require the configured strike/shareholder role
- [ ] Discord admin alone does not grant shareholder strike access

The LAN dashboard is HTTP unless placed behind a reverse proxy. Do not expose it publicly without HTTPS.
