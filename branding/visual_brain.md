# Durent Support — Visual Brain

Built following the "Grow with Alex" Brain System — Visual Brain Builder step, on top of `branding/foundation_document.md`. Once approved, paste this whole file (plus the Foundation Document) as the instructions for a dedicated Claude Project — every chat in that project then has full brand context automatically.

Colours and type are read from the existing wordmark (`Durent_ transparant-black.PNG`) and the existing event-proposal deck's described visual system (`branding/brand_identity.md` §1.4), not invented from scratch — Durent already has a working visual system, this formalizes it rather than replacing it. Exact hex values and the font pairing are **[assume] — best-fit approximations**, since the source deck's actual asset file wasn't directly inspectable in this environment. Swap in exact values from the real deck/site if they differ.

---

## Colour System

**Palette:**
- **Near-Black Navy `#0B0F14`** — Primary dark background. Matches the wordmark ink and the proposal deck's dark/moody backdrops. Used on hook slides, film-segment material, and price/CTA cards.
- **Champagne Gold `#C9A227`** — Headlines, emphasis, price highlights, CTA accents. The one restrained "premium" signal already in use in the proposal deck — keep it rare and precise, never a fill background.
- **Warm Off-White `#F7F5F1`** — Primary light background / body text on dark. No pure white anywhere.
- **Warm Stone `#EDEAE3`** — Secondary/alternate light background for carousels.
- **Slate Gray `#7A8288`** — Secondary text, captions, handle/labels.
- **Bright Amber `#D9B54A`** — CTA button fills only, slightly brighter than the gold accent so buttons read as tappable.

**Usage rules:**
- Dark navy is the default background for hook slides, film-segment material, and any price/contact card (glassmorphism style, per the existing deck).
- Gold is sparing — headlines, price numbers, one accent line. Never a background fill.
- Event-segment carousels alternate dark-navy and warm-stone slides (see Visual Rhythm).
- Image overlays: dark navy at 50–60% opacity over photography, to hold text legibility without flattening the image.

---

## Typography

**Hierarchy:**
- **Headlines:** Plus Jakarta Sans, ExtraBold (800). Title case for event-segment material, ALL CAPS for hook slides. Max 8 words on a hook.
- **Subheadings:** Plus Jakarta Sans, SemiBold (600).
- **Body text:** Inter, Regular/Medium (400/500) — clean, legible in both Bahasa Indonesia and English, holds up in small price-table text.
- **CTA text:** Plus Jakarta Sans, Bold (700), short.

*Why this pairing:* the existing wordmark is a bold, rounded, confident geometric sans. Plus Jakarta Sans carries the same confident-but-approachable geometry for headlines without trying to clone a custom logotype, and Inter is the standard, highly legible workhorse for dense bilingual body copy (specs, price tables).

**Text rules:**
- Hook slides: max 8 words. Body/value slides: max 15 words.
- Every slide combines at least 2 fonts (headline + body, or headline + accent).
- Minimum 32px body / 48px headline on carousels.
- Always test contrast against the navy/stone backgrounds — no gray-on-navy body text below the minimum sizes above.

---

## Imagery Direction

**Aesthetic:** Cinematic, dark, moody — stadium floodlights, venue and city-skyline photography at dusk, desaturated grading. Product-cutout photography (tents, chairs, AC units, sound gear, HT radios) laid over the mood backgrounds. For the event ICP specifically: **real on-site documentation beats styled stock** — proof of execution (crew actually setting up, a tenda actually standing, an actual past event) reads as more trustworthy than posed photography, because reliability is the actual sale.

**By slide position:**
- Hook: dramatic wide venue/skyline shot, high visual tension.
- Value slides: calmer — product cutouts or real job-site photos, more negative space.
- Tension slide: back to drama — the cost of *not* coordinating (chaos, last-minute scrambling).
- CTA slide: no photo. Navy background, gold glassmorphism price/contact card.

**Colour treatment:** Desaturated, cool-to-neutral grade. Dark gradient overlay top-to-bottom, 50–60% opacity, for text legibility.

**Text placement on images:** Upper third or left-aligned. Never over the product cutout itself — the product is the proof, don't cover it.

**Off-brand — never use:** Bright neon colour, cartoon illustration, generic smiling corporate stock photography, cluttered motivational-poster layouts, or any visual language that reads "tech platform / SaaS dashboard" — Durent sells a coordinated physical operation, not software.

---

## Layout and Composition

**Dimensions:** Instagram feed 1080×1080, carousels/portrait 1080×1350, stories 1080×1920. Proposal decks stay in their existing landscape/A4 package-table format — don't force that format into the social system.

**Text positioning:** Default upper third, left-aligned. Hook slides: centred, large. CTA slides: centred vertically and horizontally.

**Margins and safe zones:** Minimum 60px padding, 80px preferred. No text in the bottom 120px (Instagram UI overlap).

**Density:** Max 3 visual elements per slide (headline + body + accent, or headline + image + accent) — mirrors the existing deck's single clean glassmorphism card per screen, not clutter. Generous negative space; if a slide feels cramped, cut something rather than shrink the type.

---

## Visual Rhythm

**Background pattern (7-slide carousel):**
1. Hook — dark navy, moody photo
2. Context — dark navy, text-only, one gold accent line
3. Value — warm stone, light, text + product cutout
4. Value — dark navy, product cutout
5. Value — warm stone, light, text-only
6. Tension — dark navy, high contrast, gold emphasis on the "not five vendors" line
7. CTA — dark navy, gold glassmorphism price/contact card

**Image pattern:** Images/product cutouts on slides 1, 3, and 4. Slides 2, 5, 6 are text-only on colour.

**Energy arc:** Bold hook (the five-vendor headache) → calm context (what usually goes wrong) → build through value (what "satu operasi" actually covers) → peak at tension (the real cost of not coordinating — last-minute no-shows, vendor fatigue) → resolve at CTA (Satu Operasi, Bukan Lima Vendor + contact).

---

## Content Architecture

**Carousel: 7 slides**
- **Slide 1 — Hook:** Max 8 words. The vendor-fatigue pain, stated flat. e.g. *"5 Vendor Buat 1 Acara. Yakin Gak Ribet?"*
- **Slide 2 — Context:** 10–15 words. What usually goes wrong when an organizer sources vendors separately.
- **Slides 3–5 — Value:** One idea per slide, 10–15 words each — what "satu operasi" actually bundles (equipment, HT/comms, crew, catering, transport), one coordinated team.
- **Slide 6 — Tension:** 8–12 words. The real cost of the status quo — chasing quotes, vendors gone quiet after the DP, last-minute scrambling.
- **Slide 7 — CTA:** *"Satu Operasi, Bukan Lima Vendor."* + contact. Designed price/contact card, never an empty slide.

Every carousel should read as a narrative — hook → problem → value → tension → CTA — not a tips dump or feature list.

**Single posts:** Bold, one-image statements — a price/package headline or a proof-of-execution photo with a one-line caption.

**Story sequences (3–4 frames):** Quick, swipeable, one idea per sequence — e.g. "3 Tanda Vendor Event Lo Bakal Bikin Masalah H-1."

---

## Brand Marks

- **Handle:** `@durentsupport` **[assume — confirm actual handle]**.
- **Position:** Bottom right, every slide, Slate Gray at 60% opacity. Full gold opacity on CTA slides.
- **Sign-off / tagline:** *"Satu Operasi, Bukan Lima Vendor."*
- **Consistent element:** Wordmark lockup — dark-on-light for web/formal docs, white-on-dark for decks/social — unchanged across both segments.

---

This is the base layer for the **One Prompt** step (`docs/agent-teams.md` territory, not repeated here) — whenever there's a source article/brief and a set of brand images ready, this Visual Brain plus the Foundation Document is what makes that output land on-brand instead of generic.
