# Sudha Square

Website for **Sudha Square**, a land advisory practice operating across
Hyderabad's growth corridors. Land acquisition, development and investment
for developers, NRIs and institutional investors.

Built with Next.js 16 (App Router), React 19, Tailwind v4, Sanity CMS,
Motion and Lenis.

---

## Running it

```bash
npm install
npm run dev          # http://localhost:3000
```

The site runs with **no configuration at all**. Until Sanity is connected it
serves built-in starter content, so every page renders correctly out of the
box. The CMS replaces that content once you connect it.

| Command | What it does |
| --- | --- |
| `npm run dev` | Development server |
| `npm run build` | Production build |
| `npm start` | Serve the production build |
| `npm run lint` | ESLint |
| `npm run typecheck` | TypeScript, no emit |
| `npm run seed` | Push starter content into Sanity (see below) |

---

## Connecting the CMS

Content is managed in Sanity. The Studio is built into this app at
**`/studio`** - there is no separate deploy. The free tier covers this site
comfortably.

The project already exists: **`pav1anhp`**, dataset `production`. A project ID
is a public identifier - it ships to the browser in `NEXT_PUBLIC_` form and is
committed in `.env.example`. The API token is not; see *Secrets* below.

1. Copy `.env.example` to `.env.local`. The Sanity project ID is already in
   it.
2. Add the write token to `.env.local`. It lives at
   [sanity.io/manage](https://sanity.io/manage) under
   **API > Tokens** (Editor permissions):

   ```
   SANITY_API_WRITE_TOKEN=sk...
   ```

3. Load the starter content once, into the empty dataset:

   ```bash
   npm run seed
   ```

   This uploads the images and writes every document. It is safe to re-run
   (documents have fixed ids and are replaced, not duplicated), but that also
   means it overwrites edits made in the Studio to those same documents. Do
   not run it casually once the client has started working.

4. Run `npm run dev`, open `/studio`, and edit.

Visiting `/studio` without a project ID set shows a setup screen rather than an
error.

**Between steps 1 and 3 the site does not break.** Once the project ID is set
but before `npm run seed` runs, the dataset is empty, and most homepage
sections would otherwise delete themselves - the components render nothing
when handed an empty list. The data layer detects an unseeded dataset (no
`siteSettings` document) and keeps serving starter content for anything the
dataset is missing, per list, so real documents you have already created show
through while the rest stays intact. After seeding, empty means empty: if the
client clears the land bank, the site shows that honestly.

### What the client can edit

| Section in the Studio | Controls |
| --- | --- |
| **Site Settings** | Phone, email, address, RERA number, WhatsApp number, social links, **the top menu**, and **the footer columns and paragraph** |
| **Home Page** | Hero copy and image, the four headline figures, the verification checklist, infrastructure drivers, process steps, and **the heading above every homepage section** (under "Section Headings") |
| **Pages** | The label, heading, intro and SEO for Land Bank, Services, Insights, Case Studies, About and Contact |
| **Land Bank** | Every parcel: extent, district, zone, zoning, deal type, status, title status, survey numbers, photos, map location, 99acres and MagicBricks links |
| **Insights** | Blog posts with rich text, cover image, category and SEO |
| **Case Studies** | Closed mandates |
| **Services** | The seven advisory services |
| **Clients** | The "Trusted by" strip |
| **Land Opportunities Briefs** | The monthly PDF |

**No copy on the site is trapped in the code.** Menus, footer links, every
section heading, every page header and all listing content are edited in the
Studio. The only strings left in the source are form field labels, button
text and the 404 page.

After publishing in the Studio, the edit is live on the next page load - see
*How content reaches the site* below. There is no webhook to configure and no
revalidation window to wait out.

Read time on articles is derived from the article body, so it can never drift
out of sync with an edited post.

Menu and footer links are validated: they must start with `/` for a page on
this site or `https://` for another, which stops dead links being saved.

---

## Enquiry email

The contact form and the brief request are delivered by email through
[Resend](https://resend.com) (free tier). Set:

```
RESEND_API_KEY=...
ENQUIRY_TO_EMAIL=desk@sudhasquare.com
ENQUIRY_FROM_EMAIL="Sudha Square <noreply@your-verified-domain.com>"
```

`ENQUIRY_FROM_EMAIL` must use a domain verified in Resend.

Without these, submissions are logged to the server console and the form
succeeds **in development only**. In production an unconfigured form fails
visibly and tells the visitor to call or use WhatsApp, rather than silently
swallowing an enquiry.

---

## Deploying

The app lives at the **repository root**, so Vercel detects it with no Root
Directory override. `vercel.json` pins the framework preset, so a deploy works
even before the project is linked in the dashboard.

### Secrets

Two rules, and nothing else to remember:

- `NEXT_PUBLIC_*` variables are **not secret**. They are compiled into the
  browser bundle. The Sanity project ID and the site URL are of this kind and
  are committed in `.env.example`.
- Everything else is a secret and belongs in `.env.local` (git-ignored) and in
  the Vercel dashboard. Never in a commit. `.gitignore` covers `.env*` with a
  single exception for `.env.example`.

A leaked `SANITY_API_WRITE_TOKEN` lets anyone rewrite the site's content, and
a leaked `RESEND_API_KEY` lets anyone send mail as the domain. If either ends
up somewhere it should not - a screenshot, a chat, a commit - revoke it in the
provider dashboard and issue a new one. Revoking is instant and free; the only
cost is pasting the replacement into `.env.local` and Vercel.

### Vercel

1. Import the GitHub repo. Framework preset resolves to Next.js; leave Root
   Directory empty.
2. Set the environment variables, for **all three** environments (Production,
   Preview, Development):

   | Variable | Value | Secret |
   | --- | --- | --- |
   | `NEXT_PUBLIC_SANITY_PROJECT_ID` | `pav1anhp` | no |
   | `NEXT_PUBLIC_SANITY_DATASET` | `production` | no |
   | `NEXT_PUBLIC_SANITY_API_VERSION` | `2026-08-20` | no |
   | `NEXT_PUBLIC_SITE_URL` | `https://sudhasquare.com` | no |
   | `SANITY_API_WRITE_TOKEN` | the Editor token | **yes** |
   | `RESEND_API_KEY` | the Resend key | **yes** |
   | `ENQUIRY_TO_EMAIL` | where enquiries land | no |
   | `ENQUIRY_FROM_EMAIL` | `Sudha Square <noreply@sudhasquare.com>` | no |

   The four `NEXT_PUBLIC_` values have working defaults in the code, so a
   deploy missing them still builds - it just points canonical URLs and share
   cards at the wrong place. The enquiry form is the one that fails loudly in
   production when unconfigured, by design.
3. Deploy, then add `sudhasquare.com` under **Settings > Domains**. Add the
   apex and let Vercel add the `www` redirect.

### DNS at Hostinger

The domain stays registered at Hostinger; only the records change. In
**hPanel > Domains > DNS / Nameservers**, keep Hostinger's nameservers and
edit the records - that way Hostinger keeps serving mail:

| Type | Name | Value |
| --- | --- | --- |
| `A` | `@` | `76.76.21.21` |
| `CNAME` | `www` | `cname.vercel-dns.com` |

Delete any existing `A` or `CNAME` on `@` and `www` that point at Hostinger
hosting, or the new records will not take. **Leave the `MX` and mail-related
`TXT` records alone** - deleting those is what breaks email.

Vercel's dashboard shows the exact values it wants when the domain is added;
if they differ from the table above, Vercel is right and this file is stale.
Propagation is usually minutes, up to 48 hours. The HTTPS certificate is
issued automatically once the records resolve.

Handing the whole domain to Vercel's nameservers also works and is less
fiddly, but then Hostinger email needs its MX records re-created on the Vercel
side. Not worth it for this site.

### Anywhere else

`npm run build && npm start` behind a reverse proxy. Node 20.9+. Nothing in
the app is host-specific.

Hostinger's *shared* web hosting cannot run this app: it needs a Node server.
Hostinger **VPS** plans can. If the plan turns out to be shared hosting, use it
for the domain and mailboxes and deploy the app to Vercel.

---

## Layout of the code

```
app/
  (site)/            public site: home, land-bank, services, insights,
                     case-studies, about, contact
  studio/            Sanity Studio, mounted at /studio
  actions.ts         server actions for the enquiry and brief forms
components/
  motion/            ScrollReveal, TextReveal, AnimatedNumber, Marquee,
                     SmoothScroll
  sections/          page sections
  site/              header, footer, page header, WhatsApp button
  ui/                buttons, labels, cards, rich text
lib/
  data.ts            every read goes through here (Sanity, else seed content)
  queries.ts         GROQ
  seed-data.ts       starter content, also the payload for `npm run seed`
sanity/
  schemas/           the content model
docs/superpowers/specs/   the approved design spec
```

### How content reaches the site

**Publishing is instant.** Every content read goes straight to Sanity,
uncached, on each request (`cache: "no-store"`, and the Sanity CDN is off).
Publish in the Studio, reload the page, the change is there. There is no
webhook to configure and no revalidation window to wait out.

The trade-off is deliberate: content pages are server-rendered per request
rather than prerendered at build. For a site of this size those queries are
small and run in parallel, and the cost is worth never having to explain why
an edit has not appeared. If traffic ever makes that matter, the place to
change it is the `query` helper in `lib/data.ts` (add `next: { tags }` and a
`revalidateTag` webhook) rather than anywhere in the pages.

**Half-finished documents will not break the site.** Content gets published
mid-edit, so the read layer treats every field as possibly absent: a parcel
with no extent shows "Extent on request" rather than throwing, a missing
heading falls back to the starter copy field by field, and a document with no
slug is skipped with a named warning in the server log (it cannot be linked
to). One incomplete parcel used to take down the entire land bank page.

### Two conventions worth knowing

**All reads go through `lib/data.ts`.** That is the only place that knows
whether Sanity is configured. Pages never touch the Sanity client directly.

**Lenis owns scrolling.** Do not add `scroll-behavior: smooth` in CSS; the
two fight each other. Anchor links are routed through Lenis in
`components/motion/SmoothScroll.tsx`. Smooth scrolling is disabled entirely
under `prefers-reduced-motion`.

---

## Design notes

Three surfaces and one accent: ivory `#F7F5F2` is paper, navy `#0B132B` is
the ground, and stone `#211D18` is a warm dark that sits at roughly navy's
value on the other side of neutral. Sections alternate by temperature rather
than by two shades of one hue, which is what keeps the interior pages from
reading as a stack of blue slabs. Gold `#C9A227` is an accent only: it marks,
it never fills.

Parcel status is encoded by **contrast, not hue**. The parcel you can buy is
a solid ivory chip, one under negotiation is solid gold, and a mandated
parcel recedes to an outline. That survives being placed over any photograph
and stays readable for a colour-blind viewer, which a green/amber/grey
traffic-light does not.

Shape scale, applied everywhere: chips 4px, cards 12px, panels 20px, and
anything interactive is a full pill.

Two typefaces: **Newsreader** for display and article text, **Manrope** for
body copy and the small uppercase labels. Labels are separated by case,
tracking and weight rather than by a third family. Tokens live in
`app/globals.css`.

Motion is deliberately quiet: one entrance animation used everywhere, a slow
hero pan, figures that count once, a header that morphs on scroll. Everything
respects `prefers-reduced-motion`.
