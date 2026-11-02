# PaulStarSeed — website

Static site. No build step, no framework, no dependencies, nothing to update.
Open `index.html` and it works.

```
website/
  index.html      the whole page (all content lives here)
  style.css       all styling
  sitemap.xml     tells search engines what to index
  robots.txt      lets search engines in, points at the sitemap
  img/
    hero.jpg      landscape hero
    portrait.jpg  Invite section
    live.jpg      spare ENCOUNTER still
    og.jpg        social share card + favicon
```

---

## Why this architecture

The brief was "best architecture for future scalability and easy maintenance." For a site
like this, that means **fewer moving parts, not more**.

| Decision | Why |
|---|---|
| No framework | Nothing to update, nothing to break. Works unchanged in ten years. |
| No build step | What you see in the file is what ships. No node_modules, no CI. |
| One HTML file | Eight songs and five sections. Splitting it would add indirection for no gain. |
| Clearly marked edit zones | `EDIT HERE` comments mark the only places content lives. |
| Semantic HTML | Better SEO and accessibility for free. |

**When to revisit:** if the site grows past ~10 pages, or gains a cart or a login, move to
**Astro** — it gives components and layouts but still ships static HTML. React only if the
site becomes genuinely interactive. See `../WEBSITE-TECH-DECISIONS.md`.

---

## Making changes

### Add a song
1. In `index.html`, find `EDIT HERE: SONGS`. Copy one `<li>`, change the title and link.
2. In `<head>`, add a matching entry to the JSON-LD `"track"` list.

Those are the only two places. The HyperFollow link format is:
`https://distrokid.com/hyperfollow/paulstarseed/<song-slug>`

### Add a video
Find `EDIT HERE: VIDEOS`. Copy a `.vid` block, swap the ID after `/embed/`.
Get it from the Short's URL: `youtube.com/shorts/THIS-PART`.

### Change colours or spacing
Everything is in the `:root` block at the top of `style.css`. Change `--gold` once and it
updates everywhere.

---

## 📬 The booking form — how it works

A static site cannot send email on its own; there's no server. The form posts to
**Formspree**, which emails you the submission.

### ✅ Already set up — 2026-10-31

| | |
|---|---|
| Form name | **PaulStarSeed Bookings** |
| Endpoint | `https://formspree.io/f/xeaargwj` |
| Emails go to | **paulstarseed1@gmail.com** (verified) |
| Submissions archive | formspree.io/forms/xeaargwj/submissions |

It's already wired into `index.html`. Nothing more to do — though it's worth submitting
the form once yourself after deploying, to confirm the email lands and isn't filtered
to spam.

### The visitor never leaves the site

A plain HTML form post hands the visitor to Formspree's own thank-you page — their
branding, their navigation, and a click needed to get back. That is a bad last impression
on the one page whose job is to turn a visitor into a booking.

So the form is submitted over **Formspree's JSON API** instead: the page intercepts the
submit, posts in the background, and swaps the form for a confirmation panel in the site's
own styling. No redirect, no Formspree branding, nothing to click to get back.

This is **progressive enhancement** — the `action` and `method` attributes are still on the
`<form>` tag. If JavaScript is off or fails to load, the browser posts normally and
Formspree's page handles it. **Do not remove those two attributes.**

Three states, all verified:

| State | What the visitor sees |
|---|---|
| Sending | Button reads "Sending…" and is disabled, so it can't be double-submitted |
| Success | Gold panel: *"Invitation received"* — form removed, focus moved to the panel for screen readers |
| Failure | Red line under the button with Formspree's reason; everything they typed is preserved and the button re-enables |

⚠️ **This depends on CAPTCHA being OFF** in the Formspree form settings (it is — checked
2026-11-02). With it on, a background submission is rejected because there's no challenge
to solve. Spam is handled instead by Formshield, Formspree's server-side filter, plus the
hidden `_gotcha` honeypot field.

Ignore the **"HTTP API — Upgrade to Professional"** toggle on that settings page. That is
for *reading* submissions programmatically. Submitting over the JSON API is free.

### ✅ End-to-end delivery test — 2026-11-02

A real submission was sent through the live site and traced the whole way:

| Step | Result |
|---|---|
| Submit on paulstarseed.com | `POST https://formspree.io/f/xeaargwj` → **200** |
| Page behaviour | Stayed on `paulstarseed.com/#invite`, confirmation panel shown, no redirect |
| Formspree archive | Recorded, marked delivered, **not** flagged as spam |
| Email | Arrived in **paulstarseed1@gmail.com → Inbox** (not spam), subject *"Ministration invitation — PaulStarSeed website"*, all fields intact |
| **Reply-To** | Replying goes to **the enquirer's address**, not `noreply@formspree.io` — so you can just hit Reply |

That last row is the one that matters day to day: when a church writes in, Reply reaches
them directly. Nothing extra to configure.

The test submission is still in the inbox and the Formspree archive; delete it whenever.

### How you get notified
An email to the inbox the Formspree account uses, every time someone submits. Submissions
are also stored in the Formspree dashboard as a backup, so nothing is lost if an email
goes to spam.

### Sending to a second address
The free tier emails **one** address. Two free ways to get it to a second:

- **Gmail filter + forward** (recommended) — Gmail Settings → Filters → new filter, from
  `formspree.io`, action: forward to your other address. Forwarding needs confirming once
  at the other address.
- **Gmail delegation** or a Google Group if more than one person should see bookings.

Formspree's paid tier (~$10/mo) supports multiple recipients directly. Not worth it until
bookings actually arrive.

### Free tier limits
**50 submissions per month.** Far beyond what a new artist site receives. The form includes
a hidden honeypot field that silently blocks most spam bots.

### ⚠️ Why the email address is NOT printed on the page

It was, briefly. It has been removed.

**Publishing an email address on a public web page reliably attracts spam.** Scraper bots
crawl pages looking for `mailto:` links and plain-text addresses, harvest them, and sell
them on. Once an address is in those lists it stays there — you cannot take it back.

The form reaches the same inbox, keeps a backup copy in Formspree, and gives bots nothing
to harvest. The page now says *"Every invitation reaches PaulStarSeed directly. You'll get
a reply."* instead.

If you ever want the address shown anyway, it's one line in `index.html` — but the
trade-off is permanent.

---

## 🔍 SEO — ranking for "PaulStarSeed"

Your advantage is real: **nobody else is competing for this name.** The job isn't to beat
rivals, it's to make Google confident this site *is* the PaulStarSeed people mean.

### What's already built in

| Feature | What it does |
|---|---|
| **JSON-LD structured data** | Declares you as a `MusicGroup` with genres, tracks, links and the ENCOUNTER performance. This is what can produce a **knowledge panel** on the right of search results. |
| `sameAs` links | Connects the site to Spotify, YouTube, YouTube Music, Instagram, Facebook and HyperFollow so Google treats them as one entity. |
| `sitemap.xml` + `robots.txt` | Tells crawlers exactly what to index. |
| Open Graph + Twitter cards | Proper preview image and text when the link is shared. |
| Semantic HTML, alt text, `aria-label` | Accessibility and ranking, same work. |
| No framework | Page loads fast. Core Web Vitals are a ranking factor. |
| Canonical URL | Prevents duplicate-content confusion. |

### What you must do after deploying

1. **Google Search Console** — search.google.com/search-console. Add the property, verify,
   submit `https://paulstarseed.github.io/sitemap.xml`. **Request indexing** on the
   homepage. Without this you wait weeks; with it, often days.
2. **Bing Webmaster Tools** — same, and it feeds other engines.
3. **Link to the site from every profile you own** — YouTube About, Instagram bio, Facebook,
   HyperFollow. Google weighs those links heavily for confirming identity.
4. **Keep the name spelled one way.** `PaulStarSeed`, everywhere. Variants split your
   search signal — which is why Facebook still saying "Paulstarseed Music" matters.
5. **Update `sitemap.xml`'s `<lastmod>`** when you make real changes.

### What actually moves the needle after that
Not keyword tricks — **more places on the internet mentioning PaulStarSeed and linking
here.** Event pages that list you, church sites, blog features, playlist write-ups. One
link from a church that hosted you beats a hundred meta-tag tweaks.

### After a custom domain
If you buy `paulstarseed.com`, find-and-replace `paulstarseed.github.io` throughout
`index.html` and `sitemap.xml`, then re-verify in Search Console. Six places total.

---

## 📊 Analytics — who is actually visiting

### ✅ Cloudflare Web Analytics — live since 2026-11-02

| | |
|---|---|
| Dashboard | dash.cloudflare.com → Observability → Analytics → **Web analytics** |
| Cloudflare account | Emmpowered4wealth@gmail.com (the one that owns the domain) |
| Site | paulstarseed.com |
| Install method | **Manual JS snippet** (bottom of `index.html`) |
| Verified | `POST https://cloudflareinsights.com/cdn-cgi/rum` → `204` |

**Why the manual snippet and not "automatic setup".** Cloudflare's automatic install
injects the beacon as traffic passes *through* Cloudflare's proxy. Our DNS records are
deliberately **DNS-only (grey cloud)** — that is what lets GitHub Pages issue and renew
its own Let's Encrypt certificate. So no traffic passes through the proxy, and automatic
injection would silently record nothing. The snippet in the page does the job instead.

⚠️ **If anyone ever switches the DNS records to proxied (orange cloud)** to "turn on
Cloudflare properly", the GitHub Pages certificate breaks and the site goes to an SSL
error. Leave them grey.

**No cookie banner needed.** Cloudflare Web Analytics sets no cookies and does not
fingerprint or track individuals, so it falls outside UK/EU consent requirements. This is
the main reason it was chosen over Google Analytics for a site this size.

### What it will and won't tell you
Page views, visits, referrers, countries, device types, and Core Web Vitals. It will
**not** tell you which individual visited, or follow them across sites. For deciding
*"did the Instagram post actually send anyone to the site?"* — which is the real question
— that is enough.

### Data takes ~30 minutes to first appear
A brand-new site shows zeros at first. That is normal, not a broken install.

---

## Deploying free on GitHub Pages

1. Create a repo named exactly **`paulstarseed.github.io`**.
2. Upload the **contents** of this folder to the repo root — `index.html` must be at the
   top level, not inside `website/`.
3. Settings → Pages → Source `Deploy from a branch`, Branch `main`, folder `/`.
4. Live at `https://paulstarseed.github.io` within minutes.

### Custom domain later
Buy `paulstarseed.com` (~£10/yr), add it in Settings → Pages, and point DNS at GitHub:

```
A     @    185.199.108.153
A     @    185.199.109.153
A     @    185.199.110.153
A     @    185.199.111.153
CNAME www  paulstarseed.github.io
```

Then tick **Enforce HTTPS**.

---

## Previewing locally

```bash
python3 -m http.server 4178 --directory website
```

Open `http://localhost:4178`.

---

## ⚠️ Before publishing

1. **Decide on the public email.** `paulstarseed1@gmail.com` appears as a fallback link.
   Publishing an address invites spam; the form alone avoids that. Your call.
2. **Submit the form once** after deploying, to confirm delivery.

## Tested

Verified at 1440×900 and 375×812.

Two layout bugs found and fixed during build:
1. **Mobile nav** — the wordmark pushed the nav links off-screen below ~400px.
2. **Invite image** — had no height constraint, so it scaled unbounded and dominated the
   section. Now a fixed 4:5 crop (1:1 on mobile) with `object-fit: cover`, capped height,
   and `position: sticky` on desktop so it stays beside the form while scrolling.

Form layout collapses to single-column on phones.
