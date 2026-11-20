# PopFlix Admin + Movie / Anime / Web Series (Cloudflare)

## Project features
- Public catalog: search, categories, ratings, details
- Secure admin login with signed HttpOnly session cookie
- Add / edit / delete titles
- Movie, Anime, Web Series fields
- Seasons, episodes, synopsis, cast, poster, trailer
- Official watch links (external links; embed links may be stored as links)
- Cloudflare Workers + Static Assets + D1 database

## Important
This starter is designed for lawful catalog/discovery. Add only authorized metadata, trailers, and rights-holder official watch links. It does not host or proxy copyrighted video. Official watch links new tab mein khulte hain. Optional legal embeds sirf YouTube / YouTube-nocookie / Vimeo player hosts se allow hote hain; arbitrary URL iframe nahi hota.

## Android se GitHub upload
1. ZIP download karke Android file manager se Extract karein.
2. GitHub.com par new repository banayein (e.g. `popflix`).
3. Repository → Add file → Upload files.
4. Extracted project ke andar ki sab files/folders upload karein. `wrangler.toml`, `package.json`, `src`, `public`, `migrations` root mein hone chahiye.
5. Commit changes karein.

## Cloudflare setup (pehli baar)
Cloudflare Dashboard mein Workers & Pages → D1 SQL Database → Create database.
Database name: `popflix-db`.
Database create hone par database ID copy karein.
GitHub repository mein `wrangler.toml` edit karke `REPLACE_WITH_YOUR_D1_DATABASE_ID` ko asli ID se replace karein.

### Secrets set karein
Cloudflare Dashboard → Workers & Pages → popflix Worker → Settings → Variables and Secrets:
- `ADMIN_USERNAME` (Secret ya Text): apna admin username
- `ADMIN_PASSWORD` (Secret): ek strong, unique password (kam se kam 16 characters)
- `SESSION_SECRET` (Secret): 32+ random characters ka secret; password se alag

Secrets kabhi GitHub code mein mat daalein. Agar dashboard secret management pehle deployment ke baad available ho, pehle deploy karein, phir secrets add karke redeploy karein.

## Database migration
Is project mein D1 table migration `migrations/0001_init.sql` hai. Cloudflare dashboard D1 → popflix-db → Console mein migration SQL ka content paste karke Execute karein. (Alternative: Wrangler CLI: `npx wrangler d1 migrations apply popflix-db --remote`.)

## Deploy (Android mobile se GitHub Actions)
Is ZIP mein `.github/workflows/deploy.yml` workflow included hai. GitHub par push/commit karne par automatic deployment hoga.

1. Cloudflare Dashboard → My Profile → API Tokens → Create Token. Workers Scripts edit aur D1 edit permissions ke saath token banayein. Token ko copy karke safe rakhein.
2. Cloudflare Dashboard se Account ID copy karein.
3. GitHub repository → Settings → Secrets and variables → Actions → New repository secret.
4. Ye 2 secrets add karein:
   - `CLOUDFLARE_API_TOKEN` = Cloudflare API token
   - `CLOUDFLARE_ACCOUNT_ID` = Cloudflare Account ID
5. `main` branch mein changes commit karein. GitHub → Actions tab mein workflow status dekhein. `workflow_dispatch` se manually bhi run kar sakte hain.
6. Pehle deploy se pehle `wrangler.toml` mein D1 database ID sahi hona chahiye.

Agar GitHub Actions fail ho, Actions tab ka error screenshot share karein. Cloudflare dashboard ka simple Pages Git integration is Worker + D1 project ke liye use na karein. Agar aapko sirf Android phone hai, next step mein GitHub Actions deployment workflow add kar sakte hain.

## Admin use
Open `https://YOUR-WORKER-DOMAIN/admin.html`.
Login with configured `ADMIN_USERNAME` and `ADMIN_PASSWORD`.
Add title:
- Type: Movie / Anime / Web Series
- Poster/trailer URLs: public HTTPS image/video page URLs
- Watch links JSON example:
  `[{"label":"Official platform","url":"https://www.example.com/title","kind":"official"},{"label":"Official trailer embed","url":"https://www.youtube-nocookie.com/embed/VIDEO_ID","kind":"embed"}]`
- Seasons JSON example:
  `[{"seasonNumber":1,"title":"Season 1","episodes":[{"episodeNumber":1,"title":"Episode 1","runtime":"24m","synopsis":"Episode summary","watchLinks":[{"label":"Official","url":"https://www.example.com/episode","kind":"official"}]}]}]`
Save. Public homepage reads records from D1 immediately.

## Custom domain
Worker deploy hone ke baad Cloudflare Dashboard → Workers & Pages → popflix → Settings / Domains & Routes → Add Custom Domain. Follow DNS prompts. If your domain DNS is elsewhere, Cloudflare may ask you to point nameservers or add records. Keep existing email-related DNS records intact.

## Security and operations notes
- Use HTTPS and strong secrets.
- Do not share admin credentials.
- Change credentials if exposed.
- Add rate limiting/WAF rules for `/api/auth/login` before public launch.
- Back up D1 regularly (D1 export/backup options in dashboard).
- Validate all metadata and watch URLs; only add authorized official destinations.
- D1 free tier has usage limits; check current Cloudflare pricing/limits before scaling.
- This starter has one admin credential configured as Worker secrets. It does not yet include multiple admins, user accounts, payments, video hosting, automatic metadata scraping, or ad integration.
