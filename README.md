# FEVER SOUNDS

> One upload. Three platforms.

Premium social media management dashboard for publishing a single video to **YouTube**, **Facebook**, and **Instagram** from one place.

## Quick start

```bash
# 1. Install dependencies
bun install   # or: npm install

# 2. Configure environment
cp .env.example .env.local
# Edit .env.local with your OAuth credentials

# 3. Initialize database
bun run db:push

# 4. Start dev server
bun run dev

# 5. Open http://localhost:3000
```

## What's included

- Premium dark glassmorphism UI (Sora + Inter, bold typography)
- Single-page dashboard with sidebar nav + mobile drawer
- Create Post flow: upload once → publish to all 3 platforms
- Per-platform metadata editors (YouTube, Facebook, Instagram)
- Content Library, Scheduled Posts, Published Posts, Analytics, Settings
- Real OAuth flows for Google + Meta (server-side)
- AES-256-GCM encrypted token storage
- Per-platform independent publish jobs (one failure doesn't cancel others)
- Dynamic redirect URI (works on localhost + preview URLs + production)

## OAuth configuration

Register these redirect URIs in each platform's developer console:

| Platform | Redirect URI |
| --- | --- |
| YouTube | `http://localhost:3000/api/oauth/youtube/callback` (or your domain) |
| Meta (FB+IG) | `http://localhost:3000/api/oauth/meta/callback` (or your domain) |

The app auto-detects the host from browser requests, so the same code works on localhost, preview URLs, and production domains. Just register each URI you'll use.

## Required environment variables

```bash
DATABASE_URL=file:./db/custom.db

# Google / YouTube (https://console.cloud.google.com)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...

# Meta (https://developers.facebook.com)
META_APP_ID=...
META_APP_SECRET=...

# Instagram video hosting (public URL where videos can be fetched)
INSTAGRAM_PUBLIC_MEDIA_BASE=https://your-cdn.example.com/videos

# Optional
FEVER_ENCRYPTION_KEY=any-long-random-string
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## API setup checklist

### YouTube (Google Cloud Console)
1. Create project at https://console.cloud.google.com
2. Enable YouTube Data API v3 + YouTube Analytics API
3. Configure OAuth consent screen (External, your email)
4. Create OAuth 2.0 Client ID (Web application)
5. Add redirect URI: `http://localhost:3000/api/oauth/youtube/callback`
6. Copy Client ID + Secret to .env.local

### Facebook (Meta for Developers)
1. Create Business app at https://developers.facebook.com
2. Add "Facebook Login" product → set redirect URI: `http://localhost:3000/api/oauth/meta/callback`
3. Add "Instagram Graph API" product
4. App settings → Basic → copy App ID + Secret to .env.local
5. App Review → request advanced access for: `pages_manage_posts`, `pages_read_engagement`, `pages_show_list`, `pages_manage_metadata`, `read_insights`

### Instagram
1. Same Meta app credentials as Facebook
2. Convert Instagram account to Business/Creator
3. Link Instagram to a Facebook Page
4. Request scopes: `instagram_basic`, `instagram_content_publish`, `instagram_manage_insights`
5. Set INSTAGRAM_PUBLIC_MEDIA_BASE for video hosting

## Tech stack

- Next.js 16 (App Router) + TypeScript 5
- Tailwind CSS 4 + shadcn/ui
- Framer Motion (animations)
- Recharts (analytics)
- Zustand (client state) + TanStack Query (server state)
- Prisma + SQLite (database)
- AES-256-GCM (token encryption)

## Project structure

```
src/
  app/
    api/                          # All API routes
      accounts/                   # Connected accounts CRUD
      analytics/                  # Real metrics sync stub
      drafts/                     # Draft CRUD
      oauth/
        youtube/{start,callback}/ # Google OAuth flow
        meta/{start,callback}/    # Meta OAuth flow (FB + IG)
      publish/                    # Publish + retry endpoints
      published/                  # Published history
      schedules/                  # Schedule CRUD
      setup/                      # Returns redirect URIs + env vars
      videos/                     # Video upload/delete
    globals.css                   # Brand design system
    layout.tsx                    # Sora + Inter fonts, QueryProvider
    page.tsx                      # Single-page dashboard shell
  components/
    brand/                        # FeverLogo, PlatformIcon
    create/                       # VideoUploader, PlatformSelector, PublishDialog
    layout/                       # Sidebar, Header
    providers.tsx                 # TanStack Query provider
    shared/                       # GlassCard, StatusBadge, EmptyState
    views/                        # All 7 dashboard views
  lib/
    crypto.ts                     # AES-256-GCM token encryption
    format.ts                     # Formatters
    hooks.ts                      # TanStack Query hooks
    platform-config.ts           # OAuth scopes + dynamic redirect URIs
    publish.ts                    # YouTube/Facebook/Instagram publish adapters
    store.ts                      # Zustand store
    types.ts                      # Shared types
prisma/schema.prisma              # Full data model
.env.example                     # Environment template
```

## Production deployment

1. Set all env vars in your hosting platform (Vercel, Railway, etc.)
2. Update `NEXT_PUBLIC_APP_URL` to your production domain
3. Register the production redirect URIs in Google + Meta consoles
4. Run `bun run build` then start the production server
5. Set up a Cloud Scheduler / cron to:
   - Process scheduled posts (query ScheduledPost where scheduledAt <= now)
   - Sync analytics (write AnalyticsSnapshot rows from each platform's API)

## License

Private build for the Fever Sounds brand.
