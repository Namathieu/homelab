# Traefik edge for namathieu.ca

Traefik terminates HTTPS for `*.namathieu.ca`. Certificates are issued by
Let's Encrypt through Cloudflare's DNS-01 challenge, so inbound port 80 is not
required for certificate issuance.

## One-time setup

1. In Cloudflare, create an API token scoped only to the `namathieu.ca` zone
   with these permissions:
   - Zone / Zone / Read
   - Zone / DNS / Edit
2. Create the local configuration and secret:

   ```sh
   cp .env.example .env
   mkdir -p secrets letsencrypt
   printf '%s' 'YOUR_CLOUDFLARE_API_TOKEN' > secrets/cloudflare_dns_api_token
   chmod 600 secrets/cloudflare_dns_api_token
   ```

3. Set `ACME_EMAIL` in `.env` to a working email address.
4. Create Cloudflare DNS records for the services you want to resolve:
   - `whoami.namathieu.ca` -> the edge server's public IP for Internet access,
     or its LAN IP for local-only access.
   - `dashboard.namathieu.ca` -> the edge server's LAN IP in local/split DNS.

   The dashboard is restricted by Traefik to `127.0.0.1/32` and
   `192.168.2.0/24`; do not publish it as an Internet-facing service.

5. Start the stack:

   ```sh
   docker compose up -d
   docker compose logs -f traefik
   ```

The API token is exposed to Traefik as a Docker Compose secret and must never
be committed. ACME certificate state is stored under `letsencrypt/` and is
also excluded from Git.
