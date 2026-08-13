# Tremco Roofing & Building Maintenance — ClearSky-OMEGA Portal

Client deployment of the ClearSky-OMEGA EnergyOS portal for
**Tremco Roofing & Building Maintenance**.

Four unlocked tools on a **30-day trial**, with a one-line switch to convert to
a paid Tier 1 account.

Branding is Tremco's own wordmark (`tremco-logo.png`), used for the portal
chrome and proposal exports, with a reversed variant for dark backgrounds.

---

## ⚠ Two things to settle before this goes live

Everything else in here is done. These two are judgement calls that need a
human, and both are cheap now and expensive later.

### 1. The domain decision

`orgId` and `allowedDomain` are both set to **`tremcoinc.com`**.

That is Tremco's staff mail domain — `tremcoroofing.com` is the public web
domain and the `info@` inbox, not where employees receive mail. Sign-in keyed
to `tremcoroofing.com` would lock out the actual pilot users.

**But `tremcoinc.com` is Tremco Incorporated — the entire company**, well over
a thousand people spanning divisions far beyond Roofing & Building Maintenance.
As configured, any of them can self-serve into this workspace and land in the
same shared portfolio. For a company this size that is a much wider gate than a
trial usually wants.

Two options, both one edit in `config.js`:

| Want | Do |
|---|---|
| Open to all of Tremco Inc | Leave as is |
| Closed pilot | Blank `allowedDomain`, list the pilot team under `allowedEmails` |

Worth asking your contact which they expect. A trial that quietly admits a
thousand strangers is a different product than the one that got sold.

### 2. Trial start date

Set to **Thu Aug 13, 2026** — today, not carried over from another tenant.
If Tremco's kickoff is later, move `startsAt` before you deploy. Days spent
waiting for onboarding are days off the evaluation.

---

## Trial

| | |
|---|---|
| Account tier | **Trial** (`tierLevel: -1`) |
| Starts | **Thu Aug 13, 2026** |
| Length | **30 days** |
| Last full day | **Fri Sep 11, 2026** |
| Expires | **Sat Sep 12, 2026, 00:00** local |
| On expiry | Banner only — access continues (`lockOnExpiry: false`) |

Banner states: blue before Aug 13, amber Aug 13 – Sep 4, red Sep 5 – 11, grey
from Sep 12.

**The 30-day length fixes a problem the 14-day tenants have.** The red "ending
soon" state is hardcoded at 7 days or fewer in `omega-brand.js`, which is a
shared file and can't be tuned per tenant. On a 14-day trial that meant an
urgent red banner for the entire back half. At 30 days it is the final week
only — which is what that banner was designed to mean. No upstream change
needed.

To harden expiry into an actual lockout, set `lockOnExpiry: true` inside the
`trial` block. Domains in `adminDomains` keep access either way.

---

## Converting to Tier 1

One line, at the top of `config.js`:

```js
var PLAN = 'trial';   // ← change to 'tier1'
```

Commit, redeploy, done. No shared file is touched, no other line in `config.js`
needs editing, and it takes effect on the next page load.

| | `'trial'` | `'tier1'` |
|---|---|---|
| `accountTier` | Trial | Standard |
| `tierLevel` | -1 | 1 |
| Countdown banner | yes | none |
| Tools unlocked | **4** | **16** |
| Tools Upgrade-badged | 29 | 17 |

A misspelled `PLAN` logs a named console error and falls back to `trial` rather
than silently producing a tenant with no tier and no tools.

### What Tier 1 actually buys them

Twelve tools on top of the trial four:

| Key | Tool |
|---|---|
| `gridatlas` | Grid Atlas |
| `sandbox` | Open a Sandbox |
| `proforma` | BESS Pro Forma |
| `valuestack` | Value Stack Calculator |
| `isocalc` | BESS ISO Calculator |
| `datacenter` | Data Center Compute Calculator |
| `signal` | OMEGA Signal |
| `interconnect` | Interconnection Screener |
| `ahj` | AHJ Approval Portal — *renders "Soon", not yet clickable* |
| `procurement` | Procurement Marketplace — *"Soon"* |
| `aggregators` | Aggregators — *"Soon"* |
| `offtakers` | AI Data Offtakers — *"Soon"* |

Worth knowing before you price it: four of those twelve are "Soon" placeholders
that no tenant can open yet. The live gain is **eight** working tools, not twelve.

Tier 2 (`tierLevel: 2`) would take it to 31, Tier 3 to all 33. Neither has a
preset here — add one to the `PLANS` map in `config.js` if you need it.

---

## What's in here

| File | Shared? | Notes |
|---|---|---|
| `index.html` | **shared** | Portal dashboard |
| `marketplace.html` | **shared** | App marketplace |
| `projects.html` | **shared** | Project list |
| `editor.html` | **shared** | BESS Site Map application |
| `omega-brand.js` | **shared** | Tenant resolution + branding |
| `omega-sso.js` | **shared** | Sign-on helper |
| `omega-terms.js` | **shared** | Terms of Service gate |
| `firestore-terms.rules` | **shared** | Rule backing the ToS gate |
| `sales-proposal.html` | **not actually shared — see below** | Proposal builder |
| `config.js` | **tenant-specific** | The main file to edit |
| `tremco-logo.png` | tenant asset | Tremco wordmark |
| `tremco-logo-white.png` | tenant asset | Reversed, for dark backgrounds |
| `omega-logo.png` | platform asset | ClearSky-OMEGA mark |

Fixes to genuinely shared files belong upstream and get copied down; never
patch them here, or this repo silently forks.

### The sales-proposal.html catch

**The previous README claimed `config.js` was the only file to edit. That was
wrong, and it would have shipped another company's logo onto Tremco's
proposals.**

`sales-proposal.html` carries a hard-coded `LOGO_OVERRIDE` near the top of its
main `<script>` block, and it sits at position 0 in the brand resolution order —
ahead of `config.js`, ahead of everything. It was pointing at the previous
tenant's logo file. Repointed here, along with three brand overrides that were
left blank:

```js
var LOGO_OVERRIDE       = 'tremco-logo.png';
var LOGO_OVERRIDE_WHITE = 'tremco-logo-white.png';
var BRAND_NAME_OVERRIDE    = 'Tremco Roofing & Building Maintenance';
var BRAND_ACCENT_OVERRIDE  = '#00AA91';
var BRAND_TAGLINE_OVERRIDE = 'ROOFING · BUILDING ENVELOPE · ENERGY';
```

The name/accent/tagline are set explicitly rather than left to host resolution.
Blank, the resolver falls through `LEGACY_BRANDS` (no `tremco` key) to the slug
convention and prints a bare lowercase-derived "Tremco" in generic ClearSky
blue on customer-facing proposals.

`LOGO_OVERRIDE_WHITE` matters more than it looks here. Tremco's wordmark sets
"Roofing & Building Maintenance" in black on transparent; the page-3 footer is
`#0F1B2A` navy. Left blank, this file reuses `LOGO_OVERRIDE` and that line
disappears on every exported proposal.

**Add this file to your new-tenant checklist.** Anyone standing up the next
deployment on the old instructions will hit the same trap.

### Leftover references to other tenants

Two shared files still name previous tenants. Neither is functional for Tremco
and neither was patched, because they are upstream files:

- `sales-proposal.html` — `LEGACY_BRANDS` is a migration shim listing ten other
  ClearSky client names. It never resolves here (the overrides above win), but
  the names are readable in the source.
- `editor.html` — two comments citing other tenant names as text-measurement
  examples.

**If this repo is going to GitHub, make it private, or strip those first.** A
client-facing repo that lists nine other clients is a conversation nobody wants
to have. Deleting the whole `LEGACY_BRANDS` map is safe *in this repo* — nothing
here reaches it — but do it as an upstream decision, not a local patch, or this
copy forks.

---

## About the logo

Tremco supplied `TRBM_logo-noTM-01.webp`, converted straight to PNG at its
native **766x219**. No image processing was needed — the source already carried
a clean alpha channel, so none of the white-keying that screenshot-derived logos
need applies here.

`tremco-logo-white.png` is the reversed variant: teal `#00AA91` preserved (it
reads well on navy), black tagline lifted to white. If Tremco's brand guidelines
specify an all-white reversed lockup instead, replace that one file — nothing
references its internals.

**Two things to know:**

**It's wide.** 3.5:1, where the previous tenant's mark was 0.83:1. The sign-in
card renders the logo at `height:88px`, so this paints about **308px wide inside
324px of usable card width** — it fits, with roughly 8px of air each side. If
anyone adds padding to `.auth-card` in `index.html`, this is the first thing to
break. Mobile (≤520px) drops it to 44px tall and is nowhere near tight.

**It's a hair under 3x.** At 219px tall the source is ~2.5x the 88px render —
crisp on ordinary screens, marginally under ideal for retina. Do not upscale
the file; that adds blur, not detail. If it reads soft, ask Tremco for the
vector original (SVG/AI/EPS) and re-export. That is the only real fix.

Because the wordmark already contains the company name **and** the tagline,
`clientName` prints the same words again directly beneath it on the sign-in
card. If that reads as redundant, shorten `clientName` to `'Tremco'` — one line
in `config.js`. It's left at the full legal name so exports and the drawing
title block carry the complete name; note that the title block auto-shrinks
long names, so the full form renders small there.

---

## Before this goes live

1. **Settle the domain question** at the top of this README. Everything else is
   reversible in a redeploy; letting the wrong population sign up is not.
2. **DNS** — add a CNAME `tremco` in the `clearskyomega.com` GoDaddy zone
   pointing at whatever target Vercel issues for this project. The hash is
   per-domain, so copy it from Vercel rather than from another tenant's record.
3. **Firebase authorized domains** — Console → Authentication → Settings. Add
   both `tremco.clearskyomega.com` **and** `tremco.vercel.app`. Missing the raw
   Vercel URL is the failure mode where the page renders fine and Google
   sign-in errors out.
4. **Firestore rules** — confirm `userOrg()` maps `@tremcoinc.com` to the
   `tremcoinc.com` org.
5. **Seed their projects** with `orgId: 'tremcoinc.com'`, or the portal
   authenticates fine and shows an empty portfolio.
6. **Run "Import / Update Applications"** in the admin console if the live
   marketplace shows fewer than 33 tools — the portal hydrates its catalog from
   the Firestore `tools` collection whenever that's non-empty, and Firestore has
   historically lagged the seed in `omega-tools.js`.

If DNS gives you `DNS_PROBE_FINISHED_NXDOMAIN` right after you add the record,
that's a cached negative response on your machine, not a zone problem — flush
with `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`, then turn
off Chrome's secure DNS at `chrome://settings/security`, which keeps a separate
cache the system flush doesn't touch.

---

## Access

Sign-in is restricted to `@tremcoinc.com` — subject to the decision at the top
of this README. If they add a second domain later, uncomment `allowedDomains`
and list the extras; they all land in the same workspace, since `orgId` is fixed
regardless of which address signs in.

`csebuilders.com` and `clearsky-usa.com` may preview and survive expiry.

To admit an individual outside address, add it to the tenant rather than
opening a whole domain:

```js
allowedEmails: ['someone@example.com']
```

---

## Tools unlocked during the trial

| Key | Tool | Category |
|---|---|---|
| `editor` | BESS Site Map | design — also pinned via `requiredTools` |
| `batterysizer` | Battery Sizer | finance |
| `sales` | Sales Proposal Builder | sales |
| `financing` | Financing Partners | marketplace |

`spatco_ev` stays hidden — it's `orgs`-restricted to another tenant and never
appears here.

Two of the keys aren't literal matches for how these tools get described in
conversation, and are worth knowing about before you edit the list:
**`batterysizer`** is the "BESS sizer" (not `isocalc` or `proforma`, both of
which also carry "BESS" in their names), and **`financing`** is the "financial
marketplace" (the other marketplace entries all render "Soon").

### The gate

From `omega-tools.js`:

```
unlocked = requiredTools.has(key)
        || unlockedTools.has(key)
        || tierLevel >= (tool.tier ?? 1)
```

Tiers are `ALL=0`, `STANDARD=1`, `DELUXE=2`, `ENTERPRISE=3`. That third clause is
the whole mechanism behind the plan switch: at `-1` nothing passes on tier and
only the explicit list counts; at `1` every tier-0 and tier-1 tool opens.

`unlockedTools` stays populated under `tier1` even though tier alone would cover
all four. It's redundant but deliberate — it means those four survive any future
retiering of a tool upstream in `omega-tools.js`.

---

## Terms of Service gate

New accounts must accept Terms of Service before the portal renders. Two shared
files carry this — both byte-identical across every tenant:

| File | Role |
|---|---|
| `omega-terms.js` | The gate: consent checkbox, terms modal, Firestore record |
| `firestore-terms.rules` | The rule that must be deployed for it to work |

**Two layers, deliberately.** The sign-up form gets a consent checkbox that
blocks account creation while unticked. But the real enforcement is a gate that
runs after authentication and before the app renders — because a checkbox on the
sign-up form would miss Google sign-in entirely (a first-time Google user never
sees that form) and would miss version bumps.

**Acceptance is recorded**, which is the part that gives it weight: uid, email,
orgId, version and a server timestamp land at `termsAcceptances/{uid}`. A
checkbox nobody stored is close to worthless in a dispute.

**Amending the terms:** bump `TERMS_VERSION` at the top of `omega-terms.js`.
Every user is re-prompted on their next load. The rule permits an update only
when the version string actually changes, so an existing acceptance can't be
silently rewritten with a fresh timestamp.

### ⚠ Deploy the rule

```
firebase deploy --only firestore:rules
```

Until `termsAcceptances` is live in Firebase the acceptance write returns
permission-denied and the gate **fails closed — nobody can sign in, on any
tenant**. That direction is deliberate (failing open would let people through
ungated), but it means a forgotten rules deploy looks like a total outage. The
modal names the missing rule when it happens. Confirm the rule appears in
Firebase Console → Firestore → Rules before calling it done.

### Not legal advice

The terms are a standard SaaS starting point covering platform IP ownership,
licence scope, use restrictions (no reverse engineering, resale, white-labelling
or competing use), customer data ownership, confidentiality, trial terms, and an
engineering-output disclaimer stating that generated site plans, one-lines and
pro formas are estimates rather than sealed engineering documents. **Have a
lawyer review before relying on any of it.** Two placeholders are marked REVIEW
in the file: governing law and venue (currently Iowa) and the formal notice
address (currently `dev@clearsky-usa.com`).
