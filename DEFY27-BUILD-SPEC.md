# DEFY'27 - Build Spec & Design Reference

**Purpose:** everything worth replicating from `defy26.vercel.app`, captured so we can rebuild from scratch without re-reverse-engineering last year's site. Volatile content (dates, prizes, FAQ copy, schedule times) is deliberately **structure-only** - outreach will hand us the real values later. Sponsor data is recorded in full because that's the part we must not lose.

**Status:** reference doc. Nothing built yet. Blockers listed in §10.

**Source:** scraped from `https://defy26.vercel.app/` on 2026-10-14 (HTML + both CSS bundles). Logos pulled to `assets/`.

---

## 1. Event identity

| | |
|---|---|
| Event | DeFy'27 - 4th installment |
| Organizer | **The DAO Community** (display name - use this, not "DAO-VIT") · socials still under the `daovitcc` handle |
| Host | VIT Chennai |
| Lineage | DeFy'25 → DeFy'26 → **DeFy'27** |
| Main event dates | **8, 9, 10 January 2027** (3 days; countdown targets day 1) |
| Tagline pattern | "Defy The Limits" / "Code. Create. Conquer." |

Naming convention to keep: site lives at `defy27.vercel.app`, Devfolio slug pattern `defy-27.devfolio.co`.

---

## 2. Colour system - EXACT tokens from last year

The theme is **neon-green on true black**. These are the real values pulled from the compiled CSS, not approximations. Reuse them verbatim so '26 and '27 look like the same franchise.

### Core palette

```css
:root {
  /* surfaces - true black, not dark grey */
  --background:        #000000;   /* page */
  --card:              #0a0a0a;   /* cards, panels */
  --border:            #222222;   /* hairlines */

  /* the signature accent */
  --accent-neon:       #39ff14;   /* THE Defy green - primary accent */
  --accent-deep:       #0d400d;   /* accent surface / filled chip bg */
  --accent-foreground: #ffffff;

  /* text */
  --fg:                #ffffff;
  --fg-muted:          #b0b0b0;

  /* semantic */
  --destructive:       #ff6b60;
}
```

### Green ramp (used for charts / stat accents / gradient stops)

| Token | Hex | Note |
|---|---|---|
| `--chart-1` | `#39ff14` | primary neon |
| `--chart-2` | `#66ff33` | |
| `--chart-3` | `#99ff66` | |
| `--chart-4` | `#ccff99` | lightest |
| `--chart-5` | `#a2ff00` | acid/lime variant |

Supporting greens also present: `#00a544`, `#008138` (deeper, used for solid buttons/success), `#0a330a` and `#0d400d` (dark green surfaces).

### Signature effects - copy these exactly, they define the look

```css
/* neon glow on primary CTA / hovered cards */
box-shadow: 0 0 30px rgba(57, 255, 20, 0.4);

/* subtle ambient glow on resting cards */
box-shadow: 0 0 30px rgba(57, 255, 20, 0.1);

/* lifted card */
box-shadow: 0 8px 32px rgba(57, 255, 20, 0.15);

/* glowing icon / logo */
filter: drop-shadow(0 0 15px rgba(57, 255, 20, 0.8));

/* faint green glass fill behind panels */
background: linear-gradient(135deg, rgba(57,255,20,0.05) 0%, rgba(57,255,20,0.02) 100%);

/* the background grid - very low contrast, sits behind whole page */
background-image:
  linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
  linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px);

/* edge fade for the sponsor marquee */
mask-image: linear-gradient(transparent 0%, #000 10% 90%, transparent 100%);
```

**Rules of thumb observed:** black is the base, green is used sparingly as accent/glow/border - never large green fills. Borders are `1px solid #39ff14` on active/hovered state, `#222` at rest. No purple/blue anywhere in the brand (the stray `#ac4bff` / `#625fff` in the bundle are vendor defaults, not used).

---

## 3. Typography

```css
--font-display: "Bebas Neue";   /* headings, hero, section titles - ALL CAPS, condensed */
--font-mono:    "Bungee";       /* countdown digits, numeric stats, accents */
--font-body:    "Geist";        /* paragraphs, FAQ, labels */
                "Geist Mono";   /* code-ish details */
```

- `Bebas Neue` is the **default** font family site-wide (`--default-font-family`), which is what gives the whole page its poster/condensed-caps feel.
- `Bungee` is the mono default - used for the countdown and big numbers.
- Loaded via `next/font` with fallback metrics (`Bebas Neue Fallback` etc.) to prevent layout shift. Keep that.
- `Outfit` appears as a `--font-display` leftover - unused, drop it.

---

## 4. Page outline - section order + anchor IDs

Single long scrolling page. Keep these anchor IDs; nav and footer both link to them.

| # | Section | Anchor ID | What it is |
|---|---|---|---|
| 1 | Navbar | - | sticky, logo left, links + CTA right |
| 2 | Hero | `#home` | big "Defy The Limits" display type, primary CTA |
| 3 | Intro | - | short "all the info you need" lead-in |
| 4 | What / Who / How | - | 3-tab explainer (Radix Tabs) |
| 5 | Event overview | - | "DEFY 27 HACKATHON" headline block |
| 6 | **Countdown** | `#timer-countdown` | live countdown - see §9 |
| 7 | Tracks | `#tracks` | "Choose Your Track" - card grid |
| 8 | Previous-year stats | `#defy26-memories` | was `#defy25-memories`; 4 stat tiles + photos |
| 9 | Timeline | `#timeline` | 4-step registration→submission flow |
| 10 | Detailed schedule | `#detailed-schedule` | day-by-day, hour-by-hour |
| 11 | Current sponsors | `#current-sponsors` | logo marquee |
| 12 | Community sponsors | `#community-sponsors` | logo marquee |
| 13 | Previous sponsors | `#previous-sponsors` | logo marquee |
| 14 | FAQs | `#faqs` | accordion |
| 15 | Footer | - | see §8 |

**Nav links (5):** Home · Tracks · Timeline · Current Sponsors · FAQs - plus a `Register Now` button.

---

## 5. Component inventory

Things last year's site had that we should rebuild:

- **Sticky navbar** with smooth-scroll anchors + mobile hamburger
- **Tabbed explainer** - Radix UI Tabs, 3 panels (What / Who / How)
- **Countdown timer** - 4 units, Bungee digits, green glow
- **Track cards** - N cards, icon + title + one-line description, green border on hover
- **Stat tiles** - big number (Bungee) + label, e.g. `500+ Participants`
- **Stepped timeline** - numbered steps with date + time + blurb
- **Schedule table** - grouped by day
- **LogoLoop marquee** (`.logoloop`) - infinite horizontal scroll, edge-faded, pause + scale on hover. Used 3× for the three sponsor tiers.
- **FAQ accordion** (`.faq-item`) - `data-open` attribute drives state, question text turns `#39ff14` when open, green left-border indicator via `:before`
- **Footer** with 2 link columns + "Powered By" logos + socials

### Stack used (worth keeping - it worked)

- **Next.js** (App Router, static export friendly)
- **Tailwind CSS v4** (CSS-first `@theme` tokens - matches the `--color-*` vars found)
- **Radix UI** primitives (tabs)
- **lucide-react** icons (~35 in use)
- **next/font** for the three families
- Deployed on **Vercel**

---

## 6. Sponsors - FULL RECORD (do not lose this)

Logos are saved locally in `assets/`. Keep the three-tier split; '26's current+community sponsors roll down into '27's "previous" tier as outreach lands new ones.

### Previous-year sponsors (8) - `assets/sponsors-previous/`

| Sponsor | Link | Logo file |
|---|---|---|
| ZKWallet | https://zkinnovations.com | `zkwalletlogo.png` |
| Gizmolab | https://www.gizmolab.io | `gizmolablogo.png` |
| MTutor | https://www.m-tutor.com/ | `mtutorlogo.png` |
| MOI Technology | https://moi.technology/ | `moilogo.png` |
| Krypcore | https://krypcore.com | `krypcorelogo.png` |
| Qoneqt | https://qoneqt.com | `qoneqtlogo.png` |
| QuickNode | https://www.quicknode.com/ | `quicknodelogo.png` |
| QuillAudits | https://www.quillaudits.com/smart-contract-audit | `quilauditslogo.png` |

### Community sponsors (3) - `assets/sponsors-community/`

| Sponsor | Logo file |
|---|---|
| Odisha DAO | `odisha-dao.png` |
| The Phoenix Guild Chennai | `phoenix-guild-chennai.png` |
| Spider NITT | `spider-nitt.png` |

### Powered by - `assets/powered-by/`

| Partner | Logo file |
|---|---|
| Devfolio | `devfolio-white.png` |
| ETHIndia | `ethindia-light.png` |

> **Carry-forward note:** for '27, the 8 above + the 3 community ones are all legitimately "previous sponsors." Decide with outreach whether community partners stay in their own tier (recommended - they're relationships, not money) or merge into previous.

---

## 7. Volatile content - structure only

Do **not** hardcode. Build each of these as a typed data file (`src/data/*.ts`) so outreach can hand us values and we change one array. Shapes below are what last year's content needed:

```ts
// tracks.ts - '26 had 3. Count is flexible.
type Track = { id: string; title: string; description: string; icon: LucideIcon }

// timeline.ts - '26 had 4 steps: reg opens → reg closes → event starts → event ends
type TimelineStep = { step: number; label: string; date: string; time: string; detail: string }

// schedule.ts - grouped by day, ~8-9 slots per day
type ScheduleDay = { day: number; date: string; slots: { time: string; activity: string }[] }

// stats.ts - '26 showed 4 tiles from '25's numbers
type Stat = { value: string; label: string }

// faqs.ts - '26 had 8
type Faq = { q: string; a: string }

// sponsors.ts
type Sponsor = { name: string; logo: string; href?: string; tier: 'current' | 'community' | 'previous' }
```

**Known-volatile, awaiting outreach:** track names, prize pool figure, team-size rule, registration open/close dates, full schedule, FAQ answers, '26 stat numbers, current sponsor list.

Keep the **copy tone**: short, punchy, imperative. ("Build decentralized financial solutions." / "Innovate without boundaries.")

---

## 8. Footer spec

- **Blurb:** "Innovate. Code. Collaborate. Join the DeFy'27 Hackathon and shape the future - one idea at a time."
- **Column - Explore:** Tracks · Sponsors · Timeline · FAQ
- **Column - Get Involved:** Register · WhatsApp Community
- **Buttons:** `Register Now` · `Shortlisted Teams` · `Join Community`
- **Powered By:** Devfolio + ETHIndia logos
- **Socials:** LinkedIn · Instagram · X - all under the `daovitcc` handle
- **Copyright:** `© 2026 DeFy'27 Hackathon. All rights reserved.`

---

## 9. Countdown spec

**Target: 8 January 2027** (first day of the 8-9-10 Jan run).

```ts
// Pin to a timezone - do NOT use a bare local-time Date string.
const EVENT_START = new Date('2027-01-08T09:00:00+05:30') // IST; confirm start hour with ops
```

Requirements:
- Units: Days · Hours · Minutes · Seconds, Bungee font, neon-green glow.
- Compute from `Date.now()` on a 1s interval; always derive from the absolute target so a backgrounded tab doesn't drift.
- Render a sensible **post-deadline state** - last year's would have gone negative. Ship an `isLive` / `hasEnded` branch from day one ("The hackathon is live" → "That's a wrap").
- SSR/hydration: render a neutral placeholder on the server, start ticking in `useEffect`, or Next will throw a hydration mismatch.
- Honour `prefers-reduced-motion` for the digit-flip animation.

---

## 10. OPEN BLOCKERS - build around these, don't block on them

Three things we don't have yet. The fix is the same for all: **drive them from a single config file with a feature flag**, so flipping them on later is a one-line change and not a refactor.

```ts
// src/config/links.ts - the ONLY place these live
export const LINKS = {
  register:        null as string | null,  // ❓ platform undecided (Devfolio? Unstop? Luma? form?)
  whatsappCommunity: null as string | null, // ❓ group not created yet
  shortlistedTeams:  null as string | null, // ❓ nothing to show until after ideation round
}
```

| Blocker | Current state | Interim behaviour |
|---|---|---|
| **Registration platform** | Undecided. '26 used Devfolio (`defy-26.devfolio.co`). | Render `Register Now` as a **disabled button reading "Registrations Opening Soon"**. Do not point it at a placeholder or a dead Devfolio slug - a 404 on the main CTA is worse than an honest disabled state. |
| **WhatsApp community link** | No group yet. | Hide the `Join Community` button entirely when `null` (don't render a dead link in the footer). |
| **Shortlisted teams** | Nothing until after the ideation round. '26 pointed this at an Instagram story. | Omit the button until the list exists. Plan for it to be a proper page/modal this year rather than an IG story link. |

**Rule:** every CTA reads its href from `LINKS`. If the value is `null`, the component renders its own fallback (disabled / hidden). That way "we got the link" = edit one line, redeploy, done.

---

## 11. Security posture

Full checklist is in [`security.md`](./security.md). Important scoping insight:

**If registration stays on a third-party platform (Devfolio or similar), this site is a static marketing page with no auth, no database, and no user input.** That collapses most of the checklist:

| security.md section | Applies to a static '27 site? |
|---|---|
| §1 Auth & Access Control | **N/A** - no accounts on our site. Becomes critical only if we self-host registration. |
| §2 Input & Data Handling | **Mostly N/A** - no forms, no uploads, no DB. Only relevant if we add a contact form or the game leaderboard (see §12). |
| §3 Infra & Secrets | **FULLY APPLIES** - this is our real surface. Security headers, HTTPS/HSTS, no secrets in repo, dependency scanning, `.env` gitignored from commit #1. |
| §4 Hackathon-specific | **N/A while registration is external** - submission/judging/prize tampering lives on Devfolio's side, not ours. |
| §5 Traffic & Availability | **APPLIES** - static + Vercel CDN gets us most of this for free, which is exactly why static is the right call. The "no 404 under load" goal is basically solved by not having a backend. |

### Do-now items (cheap, ~30 min total)

- [ ] `.gitignore` with `.env*` **in the first commit**
- [ ] Security headers in `next.config.ts`: CSP, HSTS, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy`
- [ ] Dependabot / `npm audit` in CI
- [ ] All outbound sponsor links: `rel="noopener noreferrer"` + `target="_blank"`
- [ ] No secrets in client bundle - remember **everything** in a Next client component ships to the browser
- [ ] Health check + uptime monitor before 3 Jan
- [ ] Load test only if we add any dynamic endpoint

### Decision gate

> **The moment we decide to self-host registrations, security.md §1, §2 and §4 go from N/A to blocking.** That is a materially bigger project (auth, DB, RBAC, rate limiting, audit logging, file uploads). Strong recommendation: **stay on Devfolio or an equivalent platform** and keep our site static.

---

## 12. Web3 game - concept proposal

Since the site is static with little interactivity, a browser game is a good way to add life and give outreach something shareable. Constraint: **fun, on-theme, zero mainnet interaction.**

### Recommended: "Defy Vault" - an in-browser smart-contract exploit CTF

You're shown a small, deliberately vulnerable Solidity contract holding fake funds. Goal: **drain the vault.** You write the attack in a browser editor and hit Run. A real EVM executes it entirely client-side.

- **Why it's the right pick:** authentically Web3 - real Solidity, real EVM semantics, real exploits - while touching no network at all. It's also a soft skills-teaser that self-selects the exact participants we want, and it ties directly to the security theme we're already invested in.
- **No mainnet:** bundle an EVM in WASM (`revm`/`ethereumjs`) with a local in-memory chain. No RPC, no wallet, no gas, no faucet, works offline.
- **Levels (3 is enough):** 1) reentrancy 2) unchecked access control / missing `onlyOwner` 3) price-oracle manipulation. Ship with a "solved it" share card.
- **Effort:** highest of the three options - budget real time for the EVM-in-browser plumbing and a Monaco editor. Build it as a separate route (`/vault`) so it can slip without blocking the main site.

### Alternative A: "Block Builder" - MEV puzzle

A mempool of pending txs drops in; drag them into a block to maximise extracted profit before the timer. Teaches ordering, sandwiching, gas priority. Pure JS, deterministic daily puzzle, very shareable score. **Best fun-per-effort of the three** - pick this if time is tight.

### Alternative B: "Rug Run" - neon endless runner

Canvas endless runner in the exact `#39ff14`-on-black theme. Dodge rug pulls, collect tokens, survive the bear market. Maximum fun, lowest build cost, essentially zero educational value. Good filler if the other two don't fit the timeline.

### The "feels like Web3 without mainnet" trick

Optional polish for any of the above: let players **connect a wallet and sign a plain message** to attach their address to a score. Message signing is free, instant, and never touches a chain - but it gives a genuine Web3 feel and a leaderboard keyed by address/ENS.

⚠️ **If we add a leaderboard, security.md §2 comes back into scope.** A client-reported score is trivially forgeable. Either keep scores local-only (`localStorage`, no backend, no attack surface) or accept that a public leaderboard needs server-side validation + rate limiting. **Recommendation: localStorage-only for v1.**

---

## 13. Pre-build checklist

- [ ] `git init` - this directory is not yet a repo
- [ ] Scaffold Next.js + Tailwind v4, port the §2 tokens into `@theme` first
- [ ] Wire the three fonts via `next/font`
- [ ] Build `src/config/links.ts` with the §10 flags **before** any CTA component
- [ ] Stub every §7 data file with `TODO: outreach` placeholders
- [ ] Drop in the §6 sponsor logos from `assets/` - this content is final and ready
- [ ] Countdown to `2027-01-08T09:00:00+05:30`, with post-deadline state
- [ ] §11 do-now security items
- [ ] Decide the game (§12) - scope it as a separate route, not a blocker
- [ ] Confirm with outreach: registration platform, WhatsApp group, track names, prize pool

---

## Appendix - what we still need from other teams

| Need | Owner | Blocks |
|---|---|---|
| Registration platform decision | Core / Ops | Primary CTA |
| WhatsApp community group | Outreach | Footer CTA |
| Current sponsor list + logos | Outreach | `#current-sponsors` |
| Prize pool figure | Core | Hero, FAQ |
| Track names + descriptions | Core | `#tracks` |
| Registration open/close dates | Ops | `#timeline` |
| Day-by-day schedule | Ops | `#detailed-schedule` |
| DeFy'26 final stats | Core | `#defy26-memories` |
| Shortlisted teams (post-ideation) | Core | Shortlisted button |
