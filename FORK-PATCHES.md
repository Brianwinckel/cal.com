# Brewlytics Fork Patches

This fork (`Brianwinckel/cal.com`, deployed to the Vercel project `cal-com`,
serving `book.brewlytics.ai`) carries a small set of intentional deviations from
upstream `cal.com`. They are **not** meant to be upstreamed and **must survive
every upstream rebase/merge**. After pulling upstream, re-verify each patch below
is still present.

Search the codebase for `BREWLYTICS FORK PATCH` to find the inline markers.

| Patch | File(s) | Why |
| --- | --- | --- |
| Pin session cookie name in `getServerSession` | `packages/features/auth/lib/getServerSession.ts` | NextAuth `getToken` auto-detection misfires on Vercel serverless TRPC routes and falls back to the non-secure cookie name, 401-ing every authed procedure. Pin `secureCookie` + `cookieName` from `WEBAPP_URL`. |
| Pare crons to 2 daily | `apps/web/vercel.json` | Vercel Hobby plan caps cron jobs. Keep only `calendar-subscriptions-cleanup` and `tasks/cleanup`. |
| Brewlytics wordmark + favicons | header logos, favicons | Replace Cal.com branding with Brewlytics wordmark / Barley icon. |
| `noindex, nofollow` on all paths | `apps/web/next.config.ts` (`headers()`) | Keep raw `book.brewlytics.ai` booking pages out of search. The canonical booking entry point is the marketing site's `/book` page, which embeds Cal via iframe; the standalone Cal pages are thin/duplicate surfaces. The `X-Robots-Tag` header does not affect iframe/embed functionality. |

## Notes on the noindex patch

- Implemented as an `X-Robots-Tag: noindex, nofollow` response header on the
  `/:path*` entry in `apps/web/next.config.ts` `headers()`, so it applies to the
  entire deployment (booking pages **and** static assets that Google Search
  Console crawled under the `brewlytics.ai` property).
- Indexing headers are independent of iframe behavior — the embed at
  `brewlytics.ai/book` (loading `book.brewlytics.ai/embed/*`) keeps working.
- If a specific Cal page ever needs to rank on its own, scope the header to a
  narrower `source` instead of `/:path*`.
