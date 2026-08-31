# Production deployment

The production host runs the Next.js service as `glow` and proxies public HTTP
traffic through Nginx. Application secrets live only in `/etc/glow/glow.env`;
they are neither committed nor uploaded by GitHub Actions.

## GitHub configuration

Create a `production` environment in the repository, then set these environment
secrets:

| Secret | Value |
| --- | --- |
| `DEPLOY_HOST` | `85.121.208.13` |
| `DEPLOY_PORT` | `22` |
| `DEPLOY_USER` | `root` (or a restricted deploy user after it is configured) |
| `DEPLOY_SSH_KEY` | a private SSH key authorized on the server |
| `SUPABASE_ACCESS_TOKEN` | Supabase personal access token used only by CI to run migrations |
| `SUPABASE_DB_PASSWORD` | production database password |
| `SUPABASE_PROJECT_REF` | `rrjwwfcwhuhwlrsjirmy` |

The deploy workflow uploads a release tarball, builds it on the host using its
production environment values, keeps `/etc/glow/glow.env` untouched, and
restarts `glow`.

Database changes belong in `supabase/migrations/`. The migration job runs before
each production deployment and Supabase records applied versions, so previously
applied migrations are skipped.

## Host setup

Copy `glow.service` to `/etc/systemd/system/glow.service` and
`nginx-glow.conf` to `/etc/nginx/sites-available/glow`, then enable both after
the production environment file has been created.
